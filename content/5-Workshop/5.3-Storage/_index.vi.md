---
title : "Xây dựng lưu trữ hệ thông"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

Trong phần này, chúng ta sẽ xây dựng lớp lưu trữ cho hệ thống **Secure AI-Driven Document Platform** bằng Amazon S3 và Amazon DynamoDB.

Các nội dung thực hiện gồm:

1. [Tạo các Amazon S3 bucket](5.3.1-create-s3/)
2. [Tạo bảng Amazon DynamoDB](5.3.2-create-dynamodb/)
3. [Tạo prefix để tổ chức tài liệu](5.3.3-create-prefix/)
4. [Cấu hình CORS cho Amazon S3](5.3.4-configure-cors/)

Sau khi hoàn thành, hệ thống sẽ có ba vùng lưu trữ tài liệu:

+ **Quarantine bucket**: nhận tài liệu mới tải lên và chờ quét.
+ **Clean bucket**: lưu tài liệu đã được xác định là an toàn.
+ **Rejected bucket**: lưu tài liệu bị phát hiện không an toàn hoặc vi phạm chính sách.

Amazon DynamoDB được sử dụng để lưu metadata, trạng thái quét malware, kết quả phân tích AI và quyết định cuối cùng của từng tài liệu.
