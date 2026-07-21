---
title : "Xóa tài liệu"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.8.3 </b> "
---

Trong bước này, chúng ta sẽ sử dụng Lambda `DeleteMyDocumentLambda` để xóa tài liệu của người dùng.

Lambda được tích hợp với route:

```text
DELETE /documents/{documentId}
```

Function không cấu hình biến môi trường nên sử dụng các giá trị mặc định trong mã nguồn:

```text
DOCUMENTS_TABLE = SecureDocuments
DOCUMENT_BUCKET = secure-ai-document-storage
```

Lambda nhận `documentId`, đọc metadata trong DynamoDB và kiểm tra `userId` của tài liệu có trùng với claim `sub` trong JWT Token hay không.

Các trạng thái đang xử lý không được phép xóa:

```text
PENDING_UPLOAD
WAITING_MALWARE_SCAN
MALWARE_SCAN_COMPLETED
AI_ANALYZING
MOVING_TO_CLEAN
MOVING_TO_REJECT
DECISION_PENDING
```

Nếu tài liệu đang ở một trong các trạng thái trên, API trả:

```json
{
  "message": "Không thể xóa tài liệu khi hệ thống đang xử lý",
  "status": "AI_ANALYZING"
}
```

với HTTP status:

```text
409 Conflict
```

Quy trình xóa gồm ba giai đoạn:

```text
Đổi trạng thái thành DELETING
→ Xóa object hoặc object version trên Amazon S3
→ Đổi trạng thái thành DELETED
```

Lambda sử dụng conditional update để tránh xóa tài liệu khi trạng thái vừa bị thay đổi bởi tiến trình khác:

```python
table.update_item(
    Key={"documentId": document_id},
    UpdateExpression="""
        SET #status = :deleting,
            deleteStartedAt = :now,
            updatedAt = :now
    """,
    ConditionExpression="""
        userId = :userId
        AND #status = :currentStatus
    """
)
```

Object cần xóa được xác định từ thuộc tính:

```text
s3Key
```

Nếu object có version ID, Lambda ưu tiên theo thứ tự:

```text
destinationVersionId
malwareObjectVersionId
objectVersionId
```

Sau khi xóa object thành công, item DynamoDB được cập nhật:

```text
status    = DELETED
deletedAt = thời gian xóa
updatedAt = thời gian xóa
```

Thuộc tính `downloadUrl` được loại bỏ nếu tồn tại.

Execution role của Lambda được cấp quyền:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadAndSoftDeleteMetadata",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:UpdateItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments"
    },
    {
      "Sid": "DeleteOwnedDocumentObjects",
      "Effect": "Allow",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteObjectVersion"
      ],
      "Resource": [
        "arn:aws:s3:::secure-ai-document-storage/quarantine/*",
        "arn:aws:s3:::secure-ai-document-storage/clean/*",
        "arn:aws:s3:::secure-ai-document-storage/reject/*"
      ]
    }
  ]
}
```

Gửi yêu cầu:

```http
DELETE https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>
Authorization: Bearer <ACCESS_TOKEN>
```

Khi xóa thành công, API trả:

```text
204 No Content
```

![Delete document successfully](/images/5-Workshop/5.8/5.8.3/5.8.3.1.png)

{{% notice note %}}
Đây là cơ chế soft delete đối với metadata. Item vẫn được giữ trong DynamoDB với trạng thái `DELETED`, nhưng object tương ứng đã bị xóa khỏi Amazon S3.
{{% /notice %}}
