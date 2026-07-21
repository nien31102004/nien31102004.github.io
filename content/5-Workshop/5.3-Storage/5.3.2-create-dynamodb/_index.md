---
title : "Create an Amazon DynamoDB Table"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

Amazon DynamoDB is used to store document metadata, scan status, and processing results for each uploaded document.

## Create the DynamoDB Table

1. Open the [Amazon DynamoDB console](https://console.aws.amazon.com/dynamodb/).
2. Choose **Tables**.
3. Choose **Create table**.
4. Enter the following information:

+ **Table name**: `secure-documents`
+ **Partition key**: `documentId`
+ **Data type**: `String`
+ **Sort key**: Not used

![Create DynamoDB table](/images/5-Workshop/5.3/5.3.2/5.3.2.1.png)

5. In the **Table settings** section, choose **Customize settings**.
6. Under **Read/write capacity settings**, select **On-demand**.
7. Keep the default encryption settings and choose **Create table**.
8. Wait until the table status changes to **Active**.

![DynamoDB table active](/images/5-Workshop/5.3/5.3.2/5.3.2.2.png)

## Create a Global Secondary Index

This index allows documents to be queried by owner and upload time.

1. Open the `secure-documents` table.
2. Select the **Indexes** tab.
3. Choose **Create index**.
4. Configure the following settings:

+ **Partition key**: `ownerId` — `String`
+ **Sort key**: `uploadedAt` — `String`
+ **Index name**: `ownerId-uploadedAt-index`
+ **Attribute projections**: `All`

5. Choose **Create index** and wait until the index status becomes **Active**.

![Create DynamoDB index](/images/5-Workshop/5.3/5.3.2/5.3.2.3.png)

## Expected Data Structure

```json
{
  "documentId": "doc-uuid",
  "ownerId": "cognito-user-sub",
  "originalName": "report.pdf",
  "contentType": "application/pdf",
  "fileSize": 125000,
  "quarantineKey": "uploads/2026/07/20/doc-uuid/report.pdf",
  "status": "SCANNING",
  "malwareStatus": "PENDING",
  "aiStatus": "PENDING",
  "decision": "PENDING",
  "uploadedAt": "2026-07-20T08:00:00Z",
  "updatedAt": "2026-07-20T08:00:00Z"
}
```

{{% notice note %}}
When creating the table, you only need to define the primary key and the Global Secondary Index. All other attributes are automatically added by the Lambda functions as documents are uploaded and processed.
{{% /notice %}}