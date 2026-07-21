---
title : "Get Document Details"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.8.2 </b> "
---

In this step, we will use the `GetMyDocumentLambda` function to retrieve the detailed information of a document.

The Lambda function is integrated with the following route:

```text
GET /documents/{documentId}
```

Open the AWS Lambda Console and select the following function:

```text
GetMyDocumentLambda
```

Configure the following environment variables:

```text
DOCUMENTS_TABLE      = SecureDocuments
DOCUMENT_BUCKET      = secure-ai-document-storage
DOWNLOAD_URL_EXPIRES = 300
```

The Lambda function retrieves the `documentId` from the path parameter:

```python
document_id = (
    event.get("pathParameters")
    or {}
).get("documentId")
```

It then reads the corresponding item from the `SecureDocuments` table using the primary key:

```python
result = table.get_item(
    Key={"documentId": document_id},
    ConsistentRead=True
)
```

The Lambda function verifies:

+ Whether the document exists.
+ Whether the document has been marked as `DELETED`.
+ Whether the document's `userId` matches the `sub` claim in the JWT token.

If the document does not exist, has been deleted, or belongs to another user, the API returns:

```json
{
  "message": "Document not found"
}
```

The following metadata fields are returned:

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

If the document has:

```text
status = SAFE
```

and the `s3Key` begins with:

```text
clean/
```

the Lambda function generates an Amazon S3 Presigned URL and includes the following fields in the response:

```text
downloadUrl
downloadUrlExpiresIn
```

The Presigned URL is valid for:

```text
300 seconds
```

If the item contains a `destinationVersionId`, the URL is generated for the corresponding version of the object in Amazon S3.

The Lambda execution role requires the following permissions:

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

Send the following test request:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>
Authorization: Bearer <ACCESS_TOKEN>
```

A successful response is similar to the following:

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
If a document belongs to another user, the Lambda function still returns **404 Not Found** to avoid revealing whether the specified `documentId` exists in the system.
{{% /notice %}}