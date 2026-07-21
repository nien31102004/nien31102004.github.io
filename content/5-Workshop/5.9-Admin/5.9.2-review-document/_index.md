---
title : "Review Documents"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.9.2 </b> "
---

In this step, we will build the functionality for viewing review history and submitting document review decisions.

The following Lambda functions are used:

```text
AdminGetDocumentReviewsLambda
AdminReviewLambda
```

The following DynamoDB tables are used:

```text
SecureDocuments
DocumentReviews
```

## View document information and review history

The `AdminGetDocumentReviewsLambda` function is integrated with the following route:

```text
GET /admin/documents/{documentId}/review
```

Environment variables:

```text
DOCUMENTS_TABLE = SecureDocuments
REVIEWS_TABLE   = DocumentReviews
```

The Lambda function performs the following operations:

```text
Verify that the JWT belongs to the Admin group
→ Read the document from SecureDocuments
→ Query the review history from DocumentReviews
→ Return the document details and review history
```

The Lambda function supports pagination using `limit` and `nextToken`. The default value of `limit` is `20`, and the maximum value is `100`.

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

Send the request:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/documents/<DOCUMENT_ID>/review
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Each review has the following structure:

```json
{
  "reviewId": "review-id",
  "documentId": "document-id",
  "decision": "APPROVE",
  "adminRiskScore": 0,
  "reason": "The document has been reviewed.",
  "reviewedBy": "admin-user-id",
  "reviewedByGroups": [
    "Admin"
  ],
  "createdAt": "2026-07-19T08:48:48Z"
}
```

## Submit a review decision

The `AdminReviewLambda` function is integrated with the following route:

```text
POST /admin/documents/{documentId}/review
```

Environment variables:

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
  "reason": "The document has been reviewed and can be safely stored."
}
```

Validation rules:

```text
decision must be APPROVE or REJECT
adminRiskScore must be between 0 and 100
reason must contain at least 10 characters
reason is limited to a maximum of 2000 characters
```

If `adminRiskScore` is not provided, the Lambda function uses `0`.

A document can only be reviewed when its status is:

```text
MANUAL_REVIEW
DECISION_PENDING
```

An administrator can only select `APPROVE` when:

```text
malwareStatus = NO_THREATS_FOUND
```

Decision mapping:

```text
APPROVE → ALLOW
REJECT  → REJECT
```

The Lambda function generates a UUID-based `reviewId` and stores the review history in the `DocumentReviews` table.

Review table key structure:

```text
Partition key: documentId
Sort key: reviewKey
```

Where:

```text
reviewKey = createdAt#reviewId
```

The Lambda function then updates the item in `SecureDocuments`:

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

Finally, the Lambda function synchronously invokes `SecureDocDecisionEngine` with the following payload:

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

Successful response:

```json
{
  "success": true,
  "message": "The admin review has been recorded and processed by the Decision Engine.",
  "documentId": "example-document-id",
  "reviewId": "example-review-id",
  "adminDecision": "APPROVE",
  "finalAction": "ALLOW",
  "decisionEngine": {}
}
```

![Review document successfully](/images/5-Workshop/5.9/5.9.2/5.9.2.1.png)

{{% notice warning %}}
The Lambda function uses conditional updates. If the document status changes while an administrator is reviewing it, the API returns `409 Conflict` to prevent newer processing results from being overwritten.
{{% /notice %}}