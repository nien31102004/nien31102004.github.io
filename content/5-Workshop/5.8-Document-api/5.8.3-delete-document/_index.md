---
title : "Delete a Document"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.8.3 </b> "
---

In this step, we will use the `DeleteMyDocumentLambda` function to delete a user's document.

The Lambda function is integrated with the following route:

```text
DELETE /documents/{documentId}
```

This function does not use environment variables. Instead, it uses the default values defined in the source code:

```text
DOCUMENTS_TABLE = SecureDocuments
DOCUMENT_BUCKET = secure-ai-document-storage
```

The Lambda function receives the `documentId`, retrieves the document metadata from Amazon DynamoDB, and verifies that the document's `userId` matches the `sub` claim in the JWT token.

Documents in the following processing states cannot be deleted:

```text
PENDING_UPLOAD
WAITING_MALWARE_SCAN
MALWARE_SCAN_COMPLETED
AI_ANALYZING
MOVING_TO_CLEAN
MOVING_TO_REJECT
DECISION_PENDING
```

If the document is currently in one of these states, the API returns:

```json
{
  "message": "The document cannot be deleted while it is being processed.",
  "status": "AI_ANALYZING"
}
```

with the following HTTP status:

```text
409 Conflict
```

The deletion workflow consists of three stages:

```text
Update status to DELETING
→ Delete the object or object version from Amazon S3
→ Update status to DELETED
```

The Lambda function uses a conditional update to prevent deleting a document if its status has changed due to another process:

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

The object to be deleted is determined using the following attribute:

```text
s3Key
```

If the object has a version ID, the Lambda function checks the following attributes in order of priority:

```text
destinationVersionId
malwareObjectVersionId
objectVersionId
```

After the object has been deleted successfully, the Amazon DynamoDB item is updated with:

```text
status    = DELETED
deletedAt = deletion timestamp
updatedAt = deletion timestamp
```

If the `downloadUrl` attribute exists, it is removed.

The Lambda execution role requires the following permissions:

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

Send the following request:

```http
DELETE https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>
Authorization: Bearer <ACCESS_TOKEN>
```

If the document is deleted successfully, the API returns:

```text
204 No Content
```

![Delete document successfully](/images/5-Workshop/5.8/5.8.3/5.8.3.1.png)

{{% notice note %}}
This implementation uses a **soft delete** mechanism for document metadata. The item remains in Amazon DynamoDB with the status `DELETED`, while the corresponding object is permanently removed from Amazon S3.
{{% /notice %}}