---
title : "Liệt kê tài liệu"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.8.1 </b> "
---

Trong bước này, chúng ta sẽ sử dụng Lambda `ListMyDocumentsLambda` để trả về danh sách tài liệu thuộc người dùng đang đăng nhập.

Lambda được tích hợp với HTTP API `secure-document-api` tại route:

```text
GET /documents
```

Mở AWS Lambda Console và chọn function:

```text
ListMyDocumentsLambda
```

Cấu hình các biến môi trường:

```text
DOCUMENTS_TABLE = SecureDocuments
USER_INDEX      = UserCreatedAtIndex
DEFAULT_LIMIT   = 20
MAX_LIMIT       = 100
```

Bảng `SecureDocuments` có Global Secondary Index `UserCreatedAtIndex` với cấu trúc:

```text
Partition key: userId
Sort key: createdAt
Status: Active
Projected attributes: All
```

Lambda lấy claim `sub` từ JWT Token và sử dụng giá trị này làm `userId`. Sau đó Lambda thực hiện `Query` trên `UserCreatedAtIndex`.

Các tham số truy vấn chính:

```python
parameters = {
    "IndexName": USER_INDEX,
    "KeyConditionExpression": Key("userId").eq(user_id),
    "ScanIndexForward": False,
    "Limit": limit
}
```

`ScanIndexForward` được đặt thành `False` để tài liệu mới nhất được trả về trước.

Lambda hỗ trợ hai query parameter:

```text
limit
nextToken
```

Trong đó:

+ `limit` quy định số lượng tài liệu tối đa trong một lần trả về.
+ Giá trị mặc định là `20`.
+ Giá trị tối đa là `100`.
+ `nextToken` được sử dụng để lấy trang dữ liệu tiếp theo.

Các tài liệu có trạng thái sau sẽ bị loại khỏi kết quả:

```text
DELETED
```

Execution role của Lambda được cấp quyền:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "QueryUserDocuments",
      "Effect": "Allow",
      "Action": [
        "dynamodb:Query"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments",
        "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments/index/UserCreatedAtIndex"
      ]
    }
  ]
}
```

Gửi yêu cầu bằng Postman:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents
Authorization: Bearer <ACCESS_TOKEN>
```

Có thể giới hạn số lượng tài liệu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents?limit=10
Authorization: Bearer <ACCESS_TOKEN>
```

Mỗi tài liệu trong kết quả có cấu trúc tương tự:

```json
{
  "documentId": "e509813c-52b4-4558-ace0-d07c18fbf74d",
  "fileName": "admin.txt",
  "contentType": "text/plain",
  "fileSize": 84,
  "status": "SAFE",
  "riskScore": 75,
  "riskLevel": "HIGH",
  "currentPrefix": "clean",
  "finalDecision": "ALLOW",
  "malwareStatus": "NO_THREATS_FOUND",
  "aiStatus": "COMPLETED",
  "createdAt": "2026-07-19T08:02:02.034970+00:00",
  "updatedAt": "2026-07-19T08:48:49.692754+00:00"
}
```

Response thành công:

```json
{
  "success": true,
  "message": "Lấy danh sách tài liệu thành công",
  "requestId": "example-request-id",
  "userId": "current-user-id",
  "documents": [],
  "count": 0,
  "pagination": {
    "limit": 20,
    "hasMore": false,
    "nextToken": null
  }
}
```

![List documents successfully](/images/5-Workshop/5.8/5.8.1/5.8.1.1.png)

{{% notice note %}}
Lambda sử dụng `Query` trên `UserCreatedAtIndex` thay vì `Scan` toàn bộ bảng để chỉ lấy tài liệu của người dùng hiện tại.
{{% /notice %}}
