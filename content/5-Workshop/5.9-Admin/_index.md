---
title : "System Administration"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9 </b> "
---

In this section, we will build the administrator APIs used to manage documents, preview documents awaiting review, record review decisions, and manage user accounts in Amazon Cognito.

These APIs are implemented using Amazon API Gateway HTTP API and AWS Lambda. Amazon Cognito JWT Authorizer is used to authenticate callers before forwarding requests to the Lambda functions.

Project API:

```text
secure-document-api
```

Stage:

```text
$default
```

Administrator group used in this project:

```text
Admin
```

Main features:

```text
GET  /admin/documents
GET  /admin/documents/{documentId}/preview-url
GET  /admin/documents/{documentId}/review
POST /admin/documents/{documentId}/review

GET  /admin/users
GET  /admin/users/{username}
POST /admin/users/{username}/disable
POST /admin/users/{username}/enable
```

Document administration workflow:

```text
Filter the document list
→ Preview the document content
→ Review the malware scan and AI analysis results
→ Submit an APPROVE or REJECT decision
→ Invoke SecureDocDecisionEngine
→ Move the document to the appropriate storage location
```

{{% notice note %}}
All administrative Lambda functions validate the `cognito:groups` claim in the JWT Token. Requests from users who do not belong to the `Admin` group are denied.
{{% /notice %}}