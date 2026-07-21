---

title : "Introduction"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
-----------------------

#### Project Introduction

Secure AI-Driven Document Platform is a secure document storage and moderation platform built on AWS. The system allows users to upload documents, track processing status, and access only documents that have passed the security inspection process.

The project is built using a Serverless architecture combined with an Event-Driven model. Components such as Amazon API Gateway, AWS Lambda, Amazon S3, Amazon SQS, and Amazon DynamoDB work together to process documents automatically without the need to maintain traditional servers.

After a user uploads a document, the system performs two main inspection processes:

* Amazon GuardDuty Malware Protection scans the document to detect malware and security threats.
* The AI model through Bedrock Mantle analyzes the document content to detect signs of fraud, Social Engineering, or high-risk content.

Based on the inspection results, the system evaluates the safety level, updates the processing status, and moves the document to the appropriate storage area.

#### Project Objectives

The project is implemented to address security issues in the process of uploading and storing documents online, including:

* Preventing documents containing malware before users can download or use them.
* Detecting content with signs of Phishing or Social Engineering using AI.
* Authenticating and authorizing users with Amazon Cognito.
* Encrypting documents during storage and transmission.
* Automating the document moderation process using events and queues.
* Storing status, risk scores, and analysis results for lookup and administration.
* Building a system with flexible scalability and optimized operating costs.

#### System Overview

In this project, the system is divided into the following main components:

* **Frontend Application** provides the interface for login, document upload, document management, and viewing inspection results. The Frontend is built with Vite and deployed as a Static Website through Amazon S3 and Amazon CloudFront.

* **Amazon Cognito** manages accounts, authenticates users, and provides JWT Tokens. The token is included in API requests to identify users and verify access permissions.

* **Amazon API Gateway** provides APIs for the Frontend, including creating Presigned URLs, retrieving the document list, viewing document details, deleting documents, retrieving scan results, and creating download links.

* **AWS Lambda** handles the system’s business logic, such as creating Presigned URLs, querying documents, processing scan results, performing AI analysis, and making the final decision for each document.

* **Amazon S3** stores documents in a single bucket and organizes them using different prefixes:

  * `quarantine/` stores documents that have just been uploaded and are waiting for inspection.
  * `clean/` stores documents that have been identified as safe.
  * `reject/` stores documents that are rejected due to detected malware or risky content.

* **Amazon SQS** receives document processing requests and inspection results. The queue helps decouple the document upload process from the scanning process, limits data loss, and supports asynchronous processing.

* **Amazon GuardDuty Malware Protection** performs malware scanning for documents uploaded to the `quarantine/` area.

* **Bedrock Mantle** invokes an AI model to analyze content extracted from PDF, TXT, or DOCX documents. The analysis results include risk level, risk score, summary content, and recommended action.

* **Amazon DynamoDB** stores document metadata, including document name, owner, storage location, processing status, malware scan results, AI results, risk score, and update time.

* **Amazon CloudWatch** stores logs and supports monitoring the activity of Lambda functions throughout the entire document processing workflow.

#### Overview of the Document Processing Flow

The document processing workflow in the system is performed through the following main steps:

1. The user logs in to the system through Amazon Cognito and receives a JWT Token.

2. The Frontend sends a request to API Gateway to obtain a Presigned URL for uploading the document.

3. Lambda generates a `documentId`, stores the initial information in DynamoDB, and returns the Presigned URL to the Frontend.

4. The Frontend uses the Presigned URL to upload the document directly to Amazon S3 under the `quarantine/` prefix.

5. After the document is successfully uploaded, Amazon S3 generates an `ObjectCreated` event and sends the document information to Amazon SQS.

6. Lambda processes the message from SQS, updates the document status, and starts the inspection process.

7. Amazon GuardDuty Malware Protection scans the document to detect malware.

8. The GuardDuty scan result is sent through Amazon EventBridge and Amazon SQS to the Lambda function that processes the result.

9. If no malware is detected, the document content is extracted and sent to the AI model through Bedrock Mantle for analysis.

10. The Lambda Decision Engine aggregates the results from GuardDuty and AI to determine the final action.

11. Safe documents are moved from the `quarantine/` prefix to `clean/`. Dangerous documents or documents with a high risk level are moved to `reject/`.

12. The final status, scan results, risk score, and processing reason are stored in Amazon DynamoDB.

13. The Frontend calls the API to retrieve the result and display the document status to the user.

#### Document Processing Statuses

During operation, a document may go through the following statuses:

* `UPLOADED`: The document has been successfully uploaded to Amazon S3.
* `SCANNING`: The document is being scanned for malware or analyzed for content.
* `AI_ANALYSIS_COMPLETED`: The AI content analysis process has been completed.
* `SAFE`: The document is safe and has been moved to the `clean/` storage area.
* `REJECTED`: The document is rejected due to detected malware or high-risk content.
* `ERROR`: The document processing process encountered an error and needs to be checked again.

#### Expected Results

After the project is completed, the system can:

* Authenticate users and protect APIs using JWT Tokens.
* Create Presigned URLs for users to upload documents directly to Amazon S3.
* Automatically trigger the inspection process after a document is uploaded.
* Detect malware using Amazon GuardDuty Malware Protection.
* Extract and analyze document content using AI.
* Provide a risk score and appropriate processing action.
* Classify documents into the safe area or rejected area.
* Store all metadata and processing results in Amazon DynamoDB.
* Allow users to view the list, status, inspection results, and download safe documents.
* Allow administrators to monitor documents, view moderation results, and perform reviews when necessary.

![Secure AI-Driven Document Platform Architecture Overview](/images/2-Proposal/kientrucaws.png)

#### Video demo
https://drive.google.com/drive/folders/107I3wMFQx0EnR0jGuWC703YX0urXKBPx

#### Link website
https://d2aoitb7o6jdem.cloudfront.net/login