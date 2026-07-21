---
title : "Triển khai Amazon CloudFront"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.10.4 </b> "
---

Tạo CloudFront distribution để phân phối bucket frontend riêng tư qua HTTPS. Dùng [Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) thay vì public Amazon S3.

## Tạo distribution

Mở [CloudFront console](https://console.aws.amazon.com/cloudfront/) và tạo distribution với:

| Thiết lập | Giá trị |
|---|---|
| Origin domain | `securedocs-frontend-<account-id>.s3.<region>.amazonaws.com` |
| Origin type | Amazon S3 bucket origin, không dùng website endpoint |
| Origin access | Origin access control settings |
| Signing behavior | Sign requests |
| Viewer protocol policy | Redirect HTTP to HTTPS |
| Allowed methods | GET, HEAD |
| Compress objects automatically | Yes |
| Default root object | `index.html` |

![cloudfront config](/images/5-Workshop/5.10/5.10.4/5.10.4.1.png)
![Origins config](/images/5-Workshop/5.10/5.10.4/5.10.4.2.png)

Tạo OAC mới khi được yêu cầu. Chọn tùy chọn sao chép hoặc cập nhật S3 bucket policy, hoặc thêm policy sau khi thay mọi placeholder:

```json
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
                    "AWS:SourceArn": "arn:aws:cloudfront::950725740411:distribution/E21XV83QYZOOMQ"
                }
            }
        }
    ]
}
```

Điều kiện `AWS:SourceArn` ngăn CloudFront distribution khác sử dụng quyền này. Giữ S3 Block Public Access ở trạng thái bật.

## Cấu hình SPA routing

Các path React Router như `/documents/123` không tồn tại dưới dạng object S3. Tạo hai custom error response trong CloudFront:

| Phản hồi origin | Response page | Phản hồi viewer | Error cache TTL |
|---|---|---|---|
| 403 | `/index.html` | 200 | 0 |
| 404 | `/index.html` | 200 | 0 |

![error pages](/images/5-Workshop/5.10/5.10.4/5.10.4.3.png)

CloudFront sẽ trả trang React đầu vào và client-side router hiển thị route được yêu cầu. JavaScript hoặc CSS thật sự bị thiếu cũng có thể đi vào fallback này, vì vậy hãy theo dõi lỗi trình duyệt và chỉ triển khai `index.html` sau khi mọi asset được tham chiếu đã tải lên thành công.

Gắn AWS-managed response headers policy cung cấp các security header thông dụng hoặc tạo policy nghiêm ngặt hơn cho domain workshop. Chỉ thêm Content Security Policy sau khi liệt kê đầy đủ đích Cognito, API Gateway và presigned request S3; policy thiếu có thể chặn đăng nhập hoặc tải lên.


## Thêm URL production vào Cognito

Chờ distribution có trạng thái **Deployed**, sau đó lưu:

```text
https://<distribution-domain>/
```

Mở cấu hình managed login của Cognito app client và thêm chính xác:

```text
Callback URL: https://<distribution-domain>/callback
Sign-out URL: https://<distribution-domain>/
```
![Managed login pages configuration](/images/5-Workshop/5.10/5.10.4/5.10.4.4.png)

Giữ URL callback và sign-out localhost nếu vẫn cần phát triển. Không dùng callback URL wildcard.

Cập nhật CORS allowed origin cho:

- HTTP API `secure-document-api`.
- Bucket quarantine dùng cho presigned upload.
- Bucket clean nếu luồng tải xuống cần browser CORS thay vì chuyển trang.

Cho phép đúng origin `https://<distribution-domain>` không có dấu gạch chéo cuối. Không thay bằng `*` cho API có xác thực.

## Kiểm thử ứng dụng đã triển khai

Mở CloudFront URL và xác nhận:

1. `/` tải qua HTTPS và truy cập trực tiếp S3 vẫn bị từ chối.
2. Đăng nhập quay về `/callback`; đăng xuất quay về `/`.
3. Refresh deep link như `/documents/<documentId>` vẫn tải ứng dụng React.
4. Người dùng thường có thể tải lên, xem trạng thái, tải xuống tài liệu được phép và xóa tài liệu của mình.
5. Người dùng thường nhận `403` từ Admin API ngay cả khi tự nhập Admin URL.
6. Quản trị viên có thể gửi quyết định kiểm duyệt và quản lý tài khoản thử nghiệm.
7. Developer tools không có lỗi mixed content, CORS, CSP, thiếu asset hoặc lộ token.

## Phát hành phiên bản mới

Với mỗi lần cập nhật frontend:

```bash
npm run build

aws s3 sync dist s3://securedocs-frontend-<account-id> \
  --delete \
  --exclude "index.html" \
  --cache-control "public,max-age=31536000,immutable"

aws s3 cp dist/index.html \
  s3://securedocs-frontend-<account-id>/index.html \
  --content-type "text/html" \
  --cache-control "no-cache,no-store,must-revalidate"

aws cloudfront create-invalidation \
  --distribution-id <distribution-id> \
  --paths "/index.html"
```

Asset có hash thường không cần invalidation. Chỉ invalidate `index.html` nhanh và ít tốn chi phí hơn `/*`; chỉ dùng invalidation rộng khi chủ động thay đổi path không có version.

Phần Frontend hoàn tất khi xác thực, mọi User API, phân quyền Admin, presigned upload, download, deep link và tái triển khai đều hoạt động qua CloudFront domain.
