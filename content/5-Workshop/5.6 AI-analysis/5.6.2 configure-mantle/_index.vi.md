---
title : "Cấu hình Bedrock Mantle"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

Bedrock Mantle được sử dụng để gửi nội dung văn bản đã trích xuất đến mô hình AI và nhận kết quả đánh giá rủi ro dưới dạng JSON.

Chuẩn bị các thông tin kết nối Mantle:

+ **Mantle Region**: `ap-northeast-1`
+ **Mantle Base URL**: `https://bedrock-mantle.ap-northeast-1.api.aws/v1`
+ **Model ID**: Model được cấp trong Bedrock Mantle.
+ **API Key**: Khóa xác thực dùng để gọi Mantle API.

Mở [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/) và chọn **Store a new secret**.



Trong phần **Secret type**, chọn **Other type of secret** và nhập:

```json
{
  "apiKey": "<BEDROCK_MANTLE_API_KEY>"
}
```
![Create secret](/images/5-Workshop/5.6/5.6.2/5.6.2.1.png)

Đặt tên cho secret:

```text
secure-doc/mantle-api-key
```

Sau đó chọn **Store** để tạo secret.

Lưu lại tên hoặc ARN của secret để cấu hình cho Lambda:

```text
secure-doc/mantle-api-key
```

Role thực thi của Lambda cần có quyền đọc secret:

```json
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "<MANTLE_SECRET_ARN>"
}
```

![Secret name](/images/5-Workshop/5.6/5.6.2/5.6.2.2.png)

{{% notice warning %}}
Không ghi trực tiếp Mantle API Key vào mã nguồn Lambda hoặc Environment variables. API Key nên được lưu trong AWS Secrets Manager để hạn chế rò rỉ thông tin xác thực.
{{% /notice %}}

{{% notice note %}}
Lambda sử dụng API tương thích với Chat Completions tại đường dẫn `/chat/completions` và yêu cầu Mantle trả về đúng một JSON object để hệ thống xử lý tự động.
{{% /notice %}}