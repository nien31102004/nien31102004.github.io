---
title : "Frontend"
date : 2024-01-01
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

Trong phần này, bạn sẽ xây dựng ứng dụng React dạng single-page application cho nền tảng tài liệu an toàn và triển khai qua Amazon S3 cùng Amazon CloudFront.

Frontend cung cấp:

- Trang đăng nhập được quản lý bởi Amazon Cognito với Authorization Code Grant và PKCE.
- Tải lên có xác thực qua `POST /upload-url`, sau đó `PUT` trực tiếp bằng presigned URL vào bucket quarantine.
- Danh sách, chi tiết, kết quả xử lý, tải xuống và xóa tài liệu.
- Trang quản trị để kiểm duyệt tài liệu và quản lý người dùng.
- Lưu trữ tĩnh riêng tư trong Amazon S3, phân phối qua HTTPS bằng CloudFront và Origin Access Control.

Thực hiện các trang theo thứ tự:

1. [Tạo ứng dụng Vite](5.10.1-create-vite/)
2. [Tích hợp xác thực và API](5.10.2-integrate-api/)
3. [Triển khai bản build lên Amazon S3](5.10.3-deploy-s3/)
4. [Triển khai Amazon CloudFront](5.10.4-deploy-cloudfront/)

Trình duyệt không nhận AWS access key và không truy cập trực tiếp các bucket dữ liệu, ngoại trừ presigned URL S3 có thời hạn ngắn. Mọi quyền API vẫn được thực thi bởi API Gateway và Lambda.

{{% notice warning %}}
Các giá trị có tên bắt đầu bằng `VITE_` sẽ được đưa vào browser bundle. Chỉ dùng chúng cho cấu hình công khai như ID, domain, region và API URL. Không đặt mật khẩu, client secret, AWS key hoặc presigned URL trong file môi trường frontend.
{{% /notice %}}
