---
title : "List Documents"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.8.1 </b> "
---

In this step, we will use the `ListMyDocumentsLambda` function to return the list of documents belonging to the currently authenticated user.

The Lambda function is integrated with the `secure-document-api` HTTP API using the following route:

```text
GET /documents
```

Open the AWS Lambda Console and select the following function:

```text
ListMyDocumentsLambda
```

Configure the following environment variables:

```text
DOCUMENTS_TABLE = SecureDocuments
USER_INDEX      = UserCreatedAtIndex
DEFAULT_LIMIT   = 20
MAX_LIMIT       = 100
```

The `SecureDocuments` table contains a Global Secondary Index named `UserCreatedAtIndex` with the following configuration:

```text
Partition key: userId
Sort key: createdAt
Status: Active
Projected attributes: All
```

The Lambda function retrieves the `sub` claim from the JWT token and uses it as the `userId`. It then performs a `Query` operation on the `UserCreatedAtIndex`.

The primary query parameters are:

```python
parameters = {
    "IndexName": USER_INDEX,
    "KeyConditionExpression": Key("userId").eq(user_id),
    "ScanIndexForward": False,
    "Limit": limit
}
```

`ScanIndexForward` is set to `False` so that the most recently uploaded documents are returned first.

The Lambda function supports the following query parameters:

```text
limit
nextToken
```

Where:

+ `limit` specifies the maximum number of documents returned in a single request.
+ The default value is `20`.
+ The maximum value is `100`.
+ `nextToken` is used to retrieve the next page of results.

Documents with the following status are excluded from the response:

```text
DELETED
```

The Lambda execution role requires the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "QueryUserDocuments",
      "Effect": "Allow",
      "Action": [
        "dynamodb:Query"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments",
        "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments/index/UserCreatedAtIndex"
      ]
    }
  ]
}
```

Send the following request using Postman:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents
Authorization: Bearer <ACCESS_TOKEN>
```

You can limit the number of returned documents:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents?limit=10
Authorization: Bearer <ACCESS_TOKEN>
```

Each document in the response has a structure similar to the following:

```json
{
  "documentId": "e509813c-52b4-4558-ace0-d07c18fbf74d",
  "fileName": "admin.txt",
  "contentType": "text/plain",
  "fileSize": 84,
  "status": "SAFE",
  "riskScore": 75,
  "riskLevel": "HIGH",
  "currentPrefix": "clean",
  "finalDecision": "ALLOW",
  "malwareStatus": "NO_THREATS_FOUND",
  "aiStatus": "COMPLETED",
  "createdAt": "2026-07-19T08:02:02.034970+00:00",
  "updatedAt": "2026-07-19T08:48:49.692754+00:00"
}
```

A successful response is similar to the following:

```json
{
  "success": true,
  "message": "Documents retrieved successfully.",
  "requestId": "example-request-id",
  "userId": "current-user-id",
  "documents": [],
  "count": 0,
  "pagination": {
    "limit": 20,
    "hasMore": false,
    "nextToken": null
  }
}
```

![List documents successfully](/images/5-Workshop/5.8/5.8.1/5.8.1.1.png)

{{% notice note %}}
The Lambda function uses a `Query` operation on the `UserCreatedAtIndex` instead of scanning the entire table, ensuring that only documents belonging to the currently authenticated user are returned.
{{% /notice %}}