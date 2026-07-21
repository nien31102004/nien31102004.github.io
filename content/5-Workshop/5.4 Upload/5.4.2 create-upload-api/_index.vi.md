---
title : "Tạo API cấp Presigned URL"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

Amazon API Gateway được sử dụng để cung cấp endpoint cho Frontend gọi Lambda và nhận Presigned URL tải tài liệu.

Mở [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/) và chọn HTTP API đã tạo cho dự án.

Trong giao diện apis chọn create api build thoe phương HTTP API 
apis name secure-document-api
nhấn create api 
![Create HTTP Api](/images/5-Workshop/5.4/5.4.2/5.4.2.1.png)

![Select API](/images/5-Workshop/5.4/5.4.2/5.4.2.2.png)

Trong thanh điều hướng, chọn **Routes**, sau đó chọn **Create**.

Cấu hình route:

+ **Method**: `POST`
+ **Resource path**: `/upload-url`

![Create route](/images/5-Workshop/5.4/5.4.2/5.4.2.3.png)

Chọn route `POST /upload-url`, sau đó chọn **Attach integration**.

Trong phần Integration, chọn:

+ **Integration type**: `Lambda function`
+ **Lambda function**: `secure-doc-generate-upload-url`
+ **Payload format version**: `2.0`

![Attach Lambda](/images/5-Workshop/5.4/5.4.2/5.4.2.4.png)

Trong phần **Authorization**, chọn JWT Authorizer đã kết nối với Amazon Cognito.
Authorization chọn Manage authorizers chọn create 
![Create Authorization](/images/5-Workshop/5.4/5.4.2/5.4.2.5.png)
![Configure authorizer](/images/5-Workshop/5.4/5.4.2/5.4.2.6.png)

Trong phần **CORS**, cho phép:

+ **Allowed origins**: `http://localhost:5173`
+ **Allowed methods**: `*`
+ **Allowed headers**: `Authorization`, `Content-Type`

Sau khi triển khai Frontend, bổ sung domain CloudFront vào danh sách Allowed origins.

![Configure CORS](/images/5-Workshop/5.4/5.4.2/5.4.2.6.png)

Kiểm tra route đã được liên kết với Lambda thành công.

{{% notice note %}}
JWT Authorizer yêu cầu Frontend gửi Access Token hoặc ID Token trong header `Authorization`. Người dùng chưa đăng nhập sẽ nhận phản hồi `Unauthorized`.
{{% /notice %}}