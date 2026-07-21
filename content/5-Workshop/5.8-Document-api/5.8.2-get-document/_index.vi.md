---
title : "Lấy thông tin tài liệu"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.8.2 </b> "
---

Trong bước này, chúng ta sẽ sử dụng Lambda `GetMyDocumentLambda` để lấy thông tin chi tiết của một tài liệu.

Lambda được tích hợp với route:

```text
GET /documents/{documentId}
```

Mở AWS Lambda Console và chọn function:

```text
GetMyDocumentLambda
```

Các biến môi trường:

```text
DOCUMENTS_TABLE     = SecureDocuments
DOCUMENT_BUCKET     = secure-ai-document-storage
DOWNLOAD_URL_EXPIRES = 300
```

Lambda nhận `documentId` từ path parameter:

```python
document_id = (
    event.get("pathParameters")
    or {}
).get("documentId")
```

Sau đó Lambda đọc item trong bảng `SecureDocuments` bằng khóa chính:

```python
result = table.get_item(
    Key={"documentId": document_id},
    ConsistentRead=True
)
```

Lambda kiểm tra:

+ Tài liệu có tồn tại hay không.
+ Tài liệu có trạng thái `DELETED` hay không.
+ Thuộc tính `userId` có trùng với claim `sub` trong JWT Token hay không.

Nếu tài liệu không tồn tại, đã bị xóa hoặc thuộc người dùng khác, API trả:

```json
{
  "message": "Không tìm thấy tài liệu"
}
```

Các trường metadata được trả về:

```text
documentId
fileName
contentType
fileSize
status
malwareStatus
aiStatus
riskScore
riskLevel
finalDecision
aiSummary
systemDecisionReason
createdAt
updatedAt
completedAt
```

Nếu tài liệu có:

```text
status = SAFE
```

và `s3Key` bắt đầu bằng:

```text
clean/
```

Lambda tạo thêm Amazon S3 Presigned URL và đưa vào response:

```text
downloadUrl
downloadUrlExpiresIn
```

Thời gian hiệu lực của URL:

```text
300 giây
```

Nếu item có `destinationVersionId`, URL được tạo cho đúng phiên bản object trong Amazon S3.

Execution role của Lambda được cấp quyền:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadDocumentMetadata",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments"
    },
    {
      "Sid": "DownloadSafeDocuments",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": "arn:aws:s3:::secure-ai-document-storage/clean/*"
    }
  ]
}
```

Gửi yêu cầu kiểm thử:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>
Authorization: Bearer <ACCESS_TOKEN>
```

Response thành công có dạng:

```json
{
  "documentId": "e509813c-52b4-4558-ace0-d07c18fbf74d",
  "fileName": "admin.txt",
  "contentType": "text/plain",
  "fileSize": 84,
  "status": "SAFE",
  "malwareStatus": "NO_THREATS_FOUND",
  "aiStatus": "COMPLETED",
  "riskScore": 75,
  "riskLevel": "HIGH",
  "finalDecision": "ALLOW",
  "aiSummary": "The document contains a suspicious URL...",
  "systemDecisionReason": "Administrator review required...",
  "createdAt": "2026-07-19T08:02:02.034970+00:00",
  "updatedAt": "2026-07-19T08:48:49.692754+00:00",
  "completedAt": "2026-07-19T08:48:49.666357+00:00",
  "downloadUrl": "https://secure-ai-document-storage.s3.amazonaws.com/...",
  "downloadUrlExpiresIn": 300
}
```

![Get document successfully](/images/5-Workshop/5.8/5.8.2/5.8.2.1.png)

{{% notice warning %}}
Khi tài liệu thuộc người dùng khác, Lambda vẫn trả `404 Not Found` để không tiết lộ rằng `documentId` đó đang tồn tại trong hệ thống.
{{% /notice %}}
