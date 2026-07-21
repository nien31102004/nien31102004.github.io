---
title : "Manage Documents"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.9.1 </b> "
---

In this step, we will build the administrative functions for managing the document list and previewing documents that are waiting for review.

The following Lambda functions are used:

```text
AdminListDocumentsLambda
secure-doc-admin-preview-url
```

## List documents by status

The `AdminListDocumentsLambda` function is integrated with the following route:

```text
GET /admin/documents
```

Environment variables:

```text
ADMIN_GROUPS    = Admin
DEFAULT_LIMIT   = 20
DOCUMENTS_TABLE = SecureDocuments
MAX_LIMIT       = 100
STATUS_INDEX    = StatusUpdatedAtIndex
```

The Lambda function only allows users who belong to the Cognito `Admin` group.

The `SecureDocuments` table contains the following Global Secondary Index:

```text
Index name: StatusUpdatedAtIndex
Partition key: status
Sort key: updatedAt
Status: Active
```

The Lambda function supports the following document statuses:

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

If the `status` query parameter is not provided, the default value is:

```text
MANUAL_REVIEW
```

Send the following request:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Filter documents by status:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents?status=SAFE
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Retrieve all supported administrative statuses:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents?status=ALL
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

The Lambda function supports pagination using `limit` and `nextToken`. The default value of `limit` is `20`, and the maximum value is `100`.

The Lambda execution role is granted the following permissions:

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

Each document in the response may contain:

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

Successful response:

```json
{
  "success": true,
  "message": "Documents retrieved successfully",
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
The value `status=ALL` is implemented by executing multiple `Query` operations for the supported administrative statuses instead of performing a full table `Scan`.
{{% /notice %}}

## Preview documents for review

The `secure-doc-admin-preview-url` Lambda function is integrated with the following route:

```text
GET /admin/documents/{documentId}/preview-url
```

Environment variables:

```text
ALLOWED_PREVIEW_STATUS  = MANUAL_REVIEW
DOCUMENTS_TABLE         = SecureDocuments
PREVIEW_URL_EXPIRATION  = 120
```

The Lambda function verifies:

```text
The JWT belongs to the Admin group
The document status is MANUAL_REVIEW or PENDING_REVIEW
The malware scan has completed
malwareStatus is NO_THREATS_FOUND or CLEAN
```

Supported file formats:

```text
PDF
TXT
DOCX
```

For PDF documents, the Lambda function generates a Presigned URL with an expiration time of `120` seconds and sets `Content-Disposition: inline` so that the browser displays the document directly.

For TXT documents, the Lambda function reads the object and attempts to decode it in the following order:

```text
UTF-8
UTF-8 BOM
Latin-1
```

For DOCX documents, the Lambda function reads the file as a ZIP archive and extracts the content from:

```text
word/document.xml
```

TXT and DOCX content is limited to a maximum of `100000` characters.

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

Send the following request:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents/<DOCUMENT_ID>/preview-url
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

PDF response:

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

TXT or DOCX response:

```json
{
  "documentId": "example-document-id",
  "fileName": "example.txt",
  "contentType": "text/plain",
  "previewType": "TEXT",
  "content": "Document content...",
  "truncated": false,
  "status": "MANUAL_REVIEW",
  "malwareStatus": "NO_THREATS_FOUND"
}
```

![Manage documents successfully](/images/5-Workshop/5.9/5.9.1/5.9.1.1.png)

{{% notice warning %}}
Although the `ALLOWED_PREVIEW_STATUS` environment variable is configured, the current implementation directly checks the `MANUAL_REVIEW` and `PENDING_REVIEW` statuses in the source code.
{{% /notice %}}