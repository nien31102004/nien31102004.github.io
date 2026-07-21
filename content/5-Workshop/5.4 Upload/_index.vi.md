---
title : "Upload"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

Trong phần này, chúng ta sẽ xây dựng chức năng tải tài liệu lên cho hệ thống **Secure AI-Driven Document Platform** bằng AWS Lambda, Amazon API Gateway và Amazon S3 Presigned URL.

Luồng tải lên được thiết kế để Frontend không cần truy cập trực tiếp bằng thông tin xác thực AWS. Sau khi người dùng đăng nhập, Frontend gửi thông tin tài liệu đến API Gateway. API sẽ gọi Lambda để kiểm tra dữ liệu, tạo `documentId`, lưu metadata ban đầu vào Amazon DynamoDB và cấp Presigned URL có thời hạn. Frontend sau đó sử dụng URL này để tải tài liệu trực tiếp lên vùng `quarantine` của Amazon S3.

Các nội dung thực hiện gồm:

1. [Tạo Lambda cấp Presigned URL](5.4.1-create-upload-lambda/)
2. [Tạo API cấp Presigned URL](5.4.2-create-upload-api/)
3. [Kiểm tra Presigned URL](5.4.3-test-presigned-url/)

Sau khi hoàn thành, hệ thống có thể:

+ Xác định người dùng thông qua JWT Token của Amazon Cognito.
+ Kiểm tra tên file, định dạng và kích thước tài liệu.
+ Tạo mã `documentId` riêng cho từng tài liệu.
+ Lưu metadata và trạng thái ban đầu vào Amazon DynamoDB.
+ Cấp Presigned URL để Frontend tải file trực tiếp lên Amazon S3.
+ Giới hạn thời gian sử dụng URL và không công khai S3 bucket.

{{% notice note %}}
Tài liệu mới được tải lên prefix `quarantine/` và chưa được xem là an toàn. Tài liệu chỉ được chuyển sang vùng lưu trữ an toàn sau khi hoàn thành quá trình quét malware, phân tích AI và đưa ra quyết định.
{{% /notice %}}