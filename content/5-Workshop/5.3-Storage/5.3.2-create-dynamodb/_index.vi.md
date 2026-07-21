---
title : "Tạo Amazon DynamoDB Table"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

Amazon DynamoDB được sử dụng để lưu metadata, trạng thái quét và kết quả xử lý của từng tài liệu.

## Tạo bảng DynamoDB

1. Mở [Amazon DynamoDB console](https://console.aws.amazon.com/dynamodb/).
2. Chọn **Tables**.
3. Chọn **Create table**.
4. Nhập các thông tin:

+ **Table name**: `secure-documents`
+ **Partition key**: `documentId`
+ **Data type**: `String`
+ **Sort key**: không sử dụng

![Create DynamoDB table](/images/5-Workshop/5.3/5.3.2/5.3.2.1.png)

5. Trong phần **Table settings**, chọn **Customize settings**.
6. Tại **Read/write capacity settings**, chọn **On-demand**.
7. Giữ cấu hình mã hóa mặc định và chọn **Create table**.
8. Chờ đến khi trạng thái bảng chuyển thành **Active**.

![DynamoDB table active](/images/5-Workshop/5.3/5.3.2/5.3.2.2.png)

## Tạo Global Secondary Index

Index này hỗ trợ truy vấn danh sách tài liệu theo người dùng và thời gian tải lên.

1. Mở bảng `secure-documents`.
2. Chọn tab **Indexes**.
3. Chọn **Create index**.
4. Cấu hình:

+ **Partition key**: `ownerId` — `String`
+ **Sort key**: `uploadedAt` — `String`
+ **Index name**: `ownerId-uploadedAt-index`
+ **Attribute projections**: `All`

5. Chọn **Create index** và chờ trạng thái **Active**.

![Create DynamoDB index](/images/5-Workshop/5.3/5.3.2/5.3.2.3.png)

## Cấu trúc dữ liệu dự kiến

```json
{
  "documentId": "doc-uuid",
  "ownerId": "cognito-user-sub",
  "originalName": "report.pdf",
  "contentType": "application/pdf",
  "fileSize": 125000,
  "quarantineKey": "uploads/2026/07/20/doc-uuid/report.pdf",
  "status": "SCANNING",
  "malwareStatus": "PENDING",
  "aiStatus": "PENDING",
  "decision": "PENDING",
  "uploadedAt": "2026-07-20T08:00:00Z",
  "updatedAt": "2026-07-20T08:00:00Z"
}
```

{{% notice note %}}
Khi tạo bảng, chỉ cần khai báo khóa chính và index. Các thuộc tính còn lại sẽ được Lambda thêm vào khi tài liệu được tải lên và xử lý.
{{% /notice %}}
