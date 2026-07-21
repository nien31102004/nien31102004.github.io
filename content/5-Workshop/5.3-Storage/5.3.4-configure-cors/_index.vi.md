---
title : "Cấu hình CORS cho Amazon S3"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.3.4 </b> "
---

CORS cho phép frontend chạy trên localhost hoặc CloudFront tải tài liệu trực tiếp lên Amazon S3 bằng Presigned URL.

## Cấu hình CORS cho Quarantine bucket

1. Mở [Amazon S3 console](https://console.aws.amazon.com/s3/).
2. Chọn bucket:

```text
securedocs-quarantine-<account-id>
```

3. Chọn tab **Permissions**.
4. Di chuyển đến phần **Cross-origin resource sharing (CORS)**.
5. Chọn **Edit**.

![Open S3 CORS configuration](/images/5-Workshop/5.3/5.3.4/5.3.4.1.png)

6. Nhập cấu hình JSON sau:

```json
[
  {
    "AllowedHeaders": [
      "*"
    ],
    "AllowedMethods": [
      "PUT",
      "GET",
      "HEAD"
    ],
    "AllowedOrigins": [
      "http://localhost:5173",
      "https://YOUR-CLOUDFRONT-DOMAIN.cloudfront.net"
    ],
    "ExposeHeaders": [
      "ETag"
    ],
    "MaxAgeSeconds": 3000
  }
]
```

7. Thay `YOUR-CLOUDFRONT-DOMAIN` bằng domain CloudFront của dự án.
8. Chọn **Save changes**.


## Cấu hình CORS cho Clean bucket

Chỉ thực hiện khi frontend cần tải hoặc xem trước tài liệu trực tiếp từ Clean bucket.

```json
[
  {
    "AllowedHeaders": [
      "*"
    ],
    "AllowedMethods": [
      "GET",
      "HEAD"
    ],
    "AllowedOrigins": [
      "http://localhost:5173",
      "https://YOUR-CLOUDFRONT-DOMAIN.cloudfront.net"
    ],
    "ExposeHeaders": [
      "ETag",
      "Content-Type"
    ],
    "MaxAgeSeconds": 3000
  }
]
```


{{% notice warning %}}
Không sử dụng `*` trong **AllowedOrigins** ở phiên bản hoàn thiện. Chỉ cho phép domain localhost dùng khi phát triển và domain CloudFront chính thức của dự án.
{{% /notice %}}

{{% notice note %}}
CORS không làm bucket trở thành public. Quyền truy cập vẫn được kiểm soát bằng IAM, bucket policy và Presigned URL.
{{% /notice %}}
