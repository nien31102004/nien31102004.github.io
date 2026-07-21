---
title : "Document API"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8 </b> "
---

In this section, we will build a set of APIs that allow users to manage the documents they have uploaded to the system.

The APIs are implemented using Amazon API Gateway HTTP API, AWS Lambda, Amazon DynamoDB, and Amazon S3. Amazon Cognito JWT Authorizer is used to authenticate users before forwarding requests to the Lambda functions.

The project API is:

```text
secure-document-api
```

The deployment stage is:

```text
$default
```

The Amazon DynamoDB table used to store document metadata is:

```text
SecureDocuments
```

The Amazon S3 bucket used to store documents is:

```text
secure-ai-document-storage
```

The following API endpoints are implemented in this section:

```text
GET    /documents
GET    /documents/{documentId}
DELETE /documents/{documentId}
GET    /documents/{documentId}/download-url
GET    /documents/{documentId}/scan-result
```

Each Lambda function retrieves the user's identity from the `sub` claim in the JWT token. Before returning document information, generating a download URL, or deleting a document, the Lambda function verifies that the document's `userId` matches the currently authenticated user.

{{% notice note %}}
Documents marked with the `DELETED` status are no longer displayed in the document list. A document can only be downloaded when its status is `SAFE`, the final decision is `ALLOW`, and the object is stored in the `clean/` prefix.
{{% /notice %}}