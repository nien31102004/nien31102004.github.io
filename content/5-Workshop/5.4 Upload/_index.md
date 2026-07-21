---
title : "Upload"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

In this section, you will build the document upload function for the **Secure AI-Driven Document Platform** using AWS Lambda, Amazon API Gateway, and Amazon S3 Presigned URLs.

The upload flow is designed so that the Frontend does not need direct access to AWS credentials. After the user signs in, the Frontend sends the document information to API Gateway. The API invokes a Lambda function to validate the request, generate a `documentId`, store the initial metadata in Amazon DynamoDB, and return a time-limited Presigned URL. The Frontend then uses this URL to upload the document directly to the `quarantine` area in Amazon S3.

The following tasks will be completed:

1. [Create the Lambda for Presigned URL generation](5.4.1-create-upload-lambda/)
2. [Create the API for Presigned URL generation](5.4.2-create-upload-api/)
3. [Test the Presigned URL](5.4.3-test-presigned-url/)

After completing this section, the system will be able to:

+ Identify the user through an Amazon Cognito JWT Token.
+ Validate the document name, file type, and file size.
+ Generate a unique `documentId` for each document.
+ Store the initial metadata and status in Amazon DynamoDB.
+ Issue a Presigned URL that allows the Frontend to upload directly to Amazon S3.
+ Limit the URL validity period without making the S3 bucket public.

{{% notice note %}}
Newly uploaded documents are stored under the `quarantine/` prefix and are not considered safe yet. A document is moved to the safe storage area only after malware scanning, AI analysis, and the final decision have been completed.
{{% /notice %}}