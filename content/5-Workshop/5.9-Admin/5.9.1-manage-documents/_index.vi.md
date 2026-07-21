---
title : "Quản lý tài liệu"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.9.1 </b> "
---

Trong bước này, chúng ta sẽ xây dựng chức năng quản trị danh sách tài liệu và xem trước nội dung của các tài liệu đang chờ đánh giá.

Hai Lambda được sử dụng:

```text
AdminListDocumentsLambda
secure-doc-admin-preview-url
```

## Liệt kê tài liệu theo trạng thái

Lambda `AdminListDocumentsLambda` được tích hợp với route:

```text
GET /admin/documents
```

Các biến môi trường:

```text
ADMIN_GROUPS    = Admin
DEFAULT_LIMIT   = 20
DOCUMENTS_TABLE = SecureDocuments
MAX_LIMIT       = 100
STATUS_INDEX    = StatusUpdatedAtIndex
```

Lambda chỉ cho phép tài khoản thuộc Cognito group `Admin`.

Bảng `SecureDocuments` có Global Secondary Index:

```text
Index name: StatusUpdatedAtIndex
Partition key: status
Sort key: updatedAt
Status: Active
```

Lambda hỗ trợ các trạng thái:

```text
ALL
MANUAL_REVIEW
SAFE
REJECTED
AI_ERROR
SCAN_ERROR
AI_ANALYSIS_COMPLETED
DECISION_PENDING
```

Nếu không truyền query parameter `status`, Lambda sử dụng `MANUAL_REVIEW`.

Gửi yêu cầu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Lọc theo trạng thái:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents?status=SAFE
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Lấy các trạng thái quản trị được hỗ trợ:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents?status=ALL
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Lambda hỗ trợ phân trang bằng `limit` và `nextToken`. Giá trị mặc định của `limit` là `20` và tối đa là `100`.

Execution role của Lambda được cấp quyền:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "QueryDocumentsByStatus",
      "Effect": "Allow",
      "Action": [
        "dynamodb:Query"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments",
        "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments/index/StatusUpdatedAtIndex"
      ]
    }
  ]
}
```

Mỗi tài liệu trong response có thể chứa:

```text
documentId
userId
fileName
contentType
fileSize
status
currentPrefix
s3Key
malwareStatus
aiStatus
modelRiskScore
modelRiskLevel
systemRiskScore
systemRiskLevel
finalRiskScore
finalRiskLevel
systemRecommendedAction
finalDecision
systemDecisionReason
aiSummary
aiEvidence
reviewStatus
reviewReasonCode
createdAt
updatedAt
completedAt
```

Response thành công:

```json
{
  "success": true,
  "message": "Lấy danh sách tài liệu theo trạng thái thành công",
  "requestedBy": {
    "adminId": "admin-user-id",
    "groups": [
      "Admin"
    ]
  },
  "filter": {
    "status": "MANUAL_REVIEW",
    "mode": "BY_STATUS"
  },
  "documents": [],
  "count": 0,
  "pagination": {
    "limit": 20,
    "hasMore": false,
    "nextToken": null
  }
}
```

{{% notice note %}}
Giá trị `status=ALL` được xử lý bằng nhiều lệnh `Query` cho các trạng thái quản trị được hỗ trợ, không sử dụng `Scan` toàn bộ bảng.
{{% /notice %}}

## Xem trước tài liệu cần đánh giá

Lambda `secure-doc-admin-preview-url` được tích hợp với route:

```text
GET /admin/documents/{documentId}/preview-url
```

Các biến môi trường:

```text
ALLOWED_PREVIEW_STATUS  = MANUAL_REVIEW
DOCUMENTS_TABLE         = SecureDocuments
PREVIEW_URL_EXPIRATION  = 120
```

Lambda kiểm tra:

```text
JWT thuộc group Admin
Tài liệu có trạng thái MANUAL_REVIEW hoặc PENDING_REVIEW
Quét mã độc đã hoàn thành
malwareStatus là NO_THREATS_FOUND hoặc CLEAN
```

Các định dạng được hỗ trợ:

```text
PDF
TXT
DOCX
```

Đối với PDF, Lambda tạo Presigned URL có thời hạn `120` giây và đặt `Content-Disposition: inline` để trình duyệt hiển thị nội dung trực tiếp.

Đối với TXT, Lambda đọc object và giải mã theo thứ tự:

```text
UTF-8
UTF-8 BOM
Latin-1
```

Đối với DOCX, Lambda đọc file dưới dạng ZIP và trích xuất nội dung từ:

```text
word/document.xml
```

Nội dung TXT và DOCX được giới hạn tối đa `100000` ký tự.

Execution role:

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
      "Sid": "ReadQuarantinePdfForPreview",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::secure-ai-document-storage/quarantine/*"
    }
  ]
}
```

Gửi yêu cầu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents/<DOCUMENT_ID>/preview-url
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Response PDF:

```json
{
  "documentId": "example-document-id",
  "fileName": "example.pdf",
  "contentType": "application/pdf",
  "previewType": "PDF",
  "previewUrl": "https://secure-ai-document-storage.s3.amazonaws.com/...",
  "expiresIn": 120,
  "status": "MANUAL_REVIEW",
  "malwareStatus": "NO_THREATS_FOUND"
}
```

Response TXT hoặc DOCX:

```json
{
  "documentId": "example-document-id",
  "fileName": "example.txt",
  "contentType": "text/plain",
  "previewType": "TEXT",
  "content": "Nội dung tài liệu...",
  "truncated": false,
  "status": "MANUAL_REVIEW",
  "malwareStatus": "NO_THREATS_FOUND"
}
```

![Manage documents successfully](/images/5-Workshop/5.9/5.9.1/5.9.1.1.png)

{{% notice warning %}}
Biến `ALLOWED_PREVIEW_STATUS` đã được cấu hình nhưng mã nguồn hiện kiểm tra trực tiếp hai trạng thái `MANUAL_REVIEW` và `PENDING_REVIEW`.
{{% /notice %}}
