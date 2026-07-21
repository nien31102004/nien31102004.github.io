---
title : "Triển khai Frontend lên Amazon S3"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.3 </b> "
---

Build ứng dụng Vite và tải các tệp tĩnh lên một bucket S3 riêng tư dành riêng cho frontend. Không dùng lại bucket quarantine, clean hoặc rejected để lưu tài nguyên website.

## Tạo bucket frontend

Mở [Amazon S3 console](https://console.aws.amazon.com/s3/) và tạo:

```text
securedocs-frontend-<account-id>
```

Nên dùng cùng Region với workshop. Giữ các thiết lập:

- **Object Ownership:** Bucket owner enforced.
- **Block Public Access:** bật cả bốn tùy chọn.
- **Bucket Versioning:** bật nếu muốn khôi phục bản triển khai cũ đơn giản hơn.
- **Default encryption:** SSE-S3 phù hợp với tài nguyên ứng dụng công khai được lưu trong bucket riêng tư; áp dụng chính sách mã hóa của tổ chức nếu có yêu cầu khác.

![structure systems](/images/5-Workshop/5.10/5.10.3/5.10.3.1.png)

Không bật S3 static website hosting. Trang tiếp theo dùng S3 REST origin thông thường cùng CloudFront Origin Access Control để bucket vẫn riêng tư.

## Tạo môi trường production

Tạo `.env.production` với cấu hình production công khai:

```text
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=<user-pool-id>
VITE_COGNITO_CLIENT_ID=<app-client-id>
VITE_COGNITO_DOMAIN=<cognito-domain>.auth.ap-southeast-1.amazoncognito.com
VITE_API_BASE_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com
```

API URL không có stage suffix khi HTTP API dùng stage `$default`. Xác nhận Cognito app client không có client secret vì browser SPA không thể bảo vệ secret.

![evn config](/images/5-Workshop/5.10/5.10.3/5.10.3.1.png)

Build và xem thử:

```bash
npm run build
npm run preview
```

Mở preview URL, kiểm tra các route chính và xác nhận Vite đã tạo `dist/index.html` cùng các tệp có hash dưới `dist/assets/`.

## Tải bản build lên S3

Dùng AWS CloudShell hoặc AWS CLI profile có quyền triển khai:

```bash
aws s3 sync dist s3://securedocs-frontend-<account-id> \
  --delete \
  --exclude "index.html" \
  --cache-control "public,max-age=31536000,immutable"

aws s3 cp dist/index.html \
  s3://securedocs-frontend-<account-id>/index.html \
  --content-type "text/html" \
  --cache-control "no-cache,no-store,must-revalidate"
```

![uploads frontend](/images/5-Workshop/5.10/5.10.3/5.10.3.3.png)

Danh tính triển khai chỉ cần:

```text
{
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::secure-doc-frontend/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::acount_id:distribution/E21XV83QYZOOMQ"
                }
            }
        }
    ]
}
```

## Xác minh bản tải lên

Trong S3 console, xác nhận bucket có `index.html` và `assets/`. Kiểm tra metadata:

- `index.html` có `Content-Type: text/html` cùng chính sách no-cache.
- JavaScript và CSS có content type phù hợp cùng cache-control dài hạn.
- Block Public Access vẫn bật.

![uploaded](/images/5-Workshop/5.10/5.10.3/5.10.3.4.png)

Mở trực tiếp S3 object URL phải nhận Access Denied. Đây là kết quả mong đợi; chỉ CloudFront distribution ở bước tiếp theo được cấp quyền đọc.

{{% notice warning %}}
Không public bucket frontend và không thêm public-read ACL hoặc bucket policy `s3:GetObject` công khai. CloudFront Origin Access Control sẽ cung cấp quyền truy cập origin riêng tư.
{{% /notice %}}
