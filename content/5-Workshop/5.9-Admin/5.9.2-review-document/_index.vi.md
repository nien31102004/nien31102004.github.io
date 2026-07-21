---
title : "Đánh giá tài liệu"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.9.2 </b> "
---

Trong bước này, chúng ta sẽ xây dựng chức năng xem lịch sử đánh giá và gửi quyết định kiểm duyệt tài liệu.

Hai Lambda được sử dụng:

```text
AdminGetDocumentReviewsLambda
AdminReviewLambda
```

Hai bảng DynamoDB:

```text
SecureDocuments
DocumentReviews
```

## Xem thông tin và lịch sử đánh giá

Lambda `AdminGetDocumentReviewsLambda` được tích hợp với route:

```text
GET /admin/documents/{documentId}/review
```

Các biến môi trường:

```text
DOCUMENTS_TABLE = SecureDocuments
REVIEWS_TABLE   = DocumentReviews
```

Lambda thực hiện:

```text
Kiểm tra JWT thuộc group Admin
→ Đọc tài liệu từ SecureDocuments
→ Query lịch sử trong DocumentReviews
→ Trả chi tiết tài liệu và danh sách review
```

Lambda hỗ trợ phân trang bằng `limit` và `nextToken`. Giá trị mặc định của `limit` là `20`, tối đa là `100`.

Execution role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadDocument",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments"
    },
    {
      "Sid": "QueryDocumentReviews",
      "Effect": "Allow",
      "Action": [
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/DocumentReviews"
    }
  ]
}
```

Gửi yêu cầu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents/<DOCUMENT_ID>/review
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Mỗi review có cấu trúc:

```json
{
  "reviewId": "review-id",
  "documentId": "document-id",
  "decision": "APPROVE",
  "adminRiskScore": 0,
  "reason": "Tài liệu đã được kiểm tra",
  "reviewedBy": "admin-user-id",
  "reviewedByGroups": [
    "Admin"
  ],
  "createdAt": "2026-07-19T08:48:48Z"
}
```

## Gửi quyết định kiểm duyệt

Lambda `AdminReviewLambda` được tích hợp với route:

```text
POST /admin/documents/{documentId}/review
```

Các biến môi trường:

```text
ADMIN_GROUP_NAME         = Admin
DECISION_ENGINE_FUNCTION = SecureDocDecisionEngine
DOCUMENTS_TABLE          = SecureDocuments
REVIEWS_TABLE            = DocumentReviews
```

Request body:

```json
{
  "decision": "APPROVE",
  "adminRiskScore": 20,
  "reason": "Tài liệu đã được kiểm tra và có thể cho phép lưu trữ"
}
```

Quy tắc kiểm tra:

```text
decision phải là APPROVE hoặc REJECT
adminRiskScore phải từ 0 đến 100
reason phải có ít nhất 10 ký tự
reason được giới hạn tối đa 2000 ký tự
```

Nếu không truyền `adminRiskScore`, Lambda sử dụng `0`.

Tài liệu chỉ được đánh giá khi trạng thái là:

```text
MANUAL_REVIEW
DECISION_PENDING
```

Quản trị viên chỉ có thể chọn `APPROVE` khi:

```text
malwareStatus = NO_THREATS_FOUND
```

Ánh xạ quyết định:

```text
APPROVE → ALLOW
REJECT  → REJECT
```

Lambda tạo `reviewId` bằng UUID và lưu lịch sử vào bảng `DocumentReviews`.

Cấu trúc khóa của bảng review:

```text
Partition key: documentId
Sort key: reviewKey
```

Trong đó:

```text
reviewKey = createdAt#reviewId
```

Sau đó Lambda cập nhật item trong `SecureDocuments`:

```text
status = DECISION_PENDING
reviewStatus
adminDecision
adminRiskScore
adminReason
reviewedBy
reviewedAt
reviewId
finalDecision
finalDecisionSource = ADMIN_REVIEW
updatedAt
```

Cuối cùng Lambda gọi đồng bộ `SecureDocDecisionEngine` với payload:

```json
{
  "documentId": "example-document-id",
  "decisionSource": "ADMIN_REVIEW",
  "action": "ALLOW",
  "reviewedBy": "admin-user-id",
  "reviewId": "review-id",
  "internalInvocation": true
}
```

Execution role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadAndUpdateSecureDocuments",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:UpdateItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments"
    },
    {
      "Sid": "WriteDocumentReviews",
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/DocumentReviews"
    },
    {
      "Sid": "InvokeDecisionEngine",
      "Effect": "Allow",
      "Action": [
        "lambda:InvokeFunction"
      ],
      "Resource": "arn:aws:lambda:ap-southeast-1:950725740411:function:SecureDocDecisionEngine"
    }
  ]
}
```

Response thành công:

```json
{
  "success": true,
  "message": "Admin review đã được ghi nhận và Decision Engine đã xử lý",
  "documentId": "example-document-id",
  "reviewId": "example-review-id",
  "adminDecision": "APPROVE",
  "finalAction": "ALLOW",
  "decisionEngine": {}
}
```

![Review document successfully](/images/5-Workshop/5.9/5.9.2/5.9.2.1.png)

{{% notice warning %}}
Lambda sử dụng conditional update. Nếu trạng thái tài liệu thay đổi trong lúc quản trị viên đang đánh giá, API trả `409 Conflict` để tránh ghi đè kết quả xử lý mới hơn.
{{% /notice %}}
