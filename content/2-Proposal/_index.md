---
title: "Project Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# SECUREDOCS AI
## An AI-Driven Secure Document Storage and Moderation Platform on AWS

### 1. Executive Summary

**SecureDocs AI** is a serverless web application on AWS that allows users to upload documents, monitor their processing status, and store them in a trusted area only after security validation has been completed.

The platform combines **Amazon GuardDuty Malware Protection for S3** for malware detection and **Amazon Bedrock Mantle** for document content analysis. Based on the results from both security layers, the **Decision Engine** automatically selects one of three outcomes:

+ `ALLOW`: The document is considered safe and is moved to the `clean/` area.
+ `REVIEW`: The document requires manual administrator review and remains in `quarantine/`.
+ `REJECT`: The document contains malware or presents a high level of risk and is moved to `reject/`.

Users are authenticated through **Amazon Cognito**, while protected API routes use an Amazon API Gateway JWT Authorizer. Document metadata, malware scan results, AI analysis results, processing states, and final decisions are stored in Amazon DynamoDB.

The platform is intended for small businesses, research teams, and internal document management systems that require an automated and scalable security review process without operating traditional servers.

### 2. Problem Statement

#### Current Problems

In many organizations, internal documents are exchanged through email, chat applications, or shared folders. This process may introduce several risks:

+ Users may upload files containing malware.
+ Documents may contain phishing, social engineering, or credential theft content.
+ Unverified documents may be stored together with trusted data.
+ Processing states and document history are not centrally tracked.
+ Administrators may not know why a document was blocked or sent for review.
+ Scaling the system may require additional servers, scanning software, and access control mechanisms.

#### Proposed Solution

SecureDocs AI uses an AWS serverless architecture to create an automated upload and document moderation workflow:

1. The user signs in through Amazon Cognito.
2. The Frontend requests a Presigned URL through Amazon API Gateway.
3. AWS Lambda generates a `documentId`, stores initial metadata in DynamoDB, and returns the Presigned URL.
4. The Frontend uploads the document directly to the Amazon S3 `quarantine/` area.
5. Amazon GuardDuty Malware Protection for S3 scans the new object.
6. GuardDuty scan results are delivered through Amazon EventBridge and Amazon SQS.
7. When the result is `NO_THREATS_FOUND`, a Lambda function extracts document content and sends it to Amazon Bedrock Mantle.
8. The Decision Engine combines the security and AI results and returns `ALLOW`, `REVIEW`, or `REJECT`.
9. The document is moved to `clean/`, moved to `reject/`, or kept in `quarantine/`.
10. The Frontend queries the API to display status, results, and processing history.

#### Benefits and Return on Investment

The proposed solution provides the following benefits:

+ Reduces manual document inspection.
+ Prevents unverified documents from entering the trusted storage area.
+ Standardizes document processing through trackable states.
+ Allows administrators to review ambiguous cases.
+ Uses a serverless architecture to reduce server maintenance.
+ Scales with the number of documents and users.
+ Provides a foundation for search, reporting, auditing, and document lifecycle management.

The main return on investment comes from reducing manual review time, lowering the risk of distributing unsafe documents, and improving traceability during security incidents.

### 3. Solution Architecture

SecureDocs AI uses a serverless, event-driven architecture and separates documents into three logical storage areas:

+ `quarantine/`: Receives new documents and holds them during security validation.
+ `clean/`: Stores documents that have been verified as safe.
+ `reject/`: Stores documents that contain malware or present a high level of risk.

![SecureDocs AI Architecture](/images/2-Proposal/securedocs-ai-architecture.png)

#### Main Processing Flow

1. **Authentication**: The user signs in through Amazon Cognito and receives a JWT Token.
2. **Upload request**: The Frontend sends file information to API Gateway.
3. **Presigned URL**: Lambda validates the file name, type, and size, creates a `documentId`, and generates a Presigned URL.
4. **Quarantine upload**: The Frontend uploads the file directly to Amazon S3 using `PUT`.
5. **Malware scanning**: GuardDuty Malware Protection for S3 scans the object.
6. **Event processing**: EventBridge and SQS deliver the scan result to Lambda.
7. **AI analysis**: Lambda extracts content from TXT, PDF, or DOCX documents and sends it to Bedrock Mantle.
8. **Decision**: The Decision Engine combines the results and selects `ALLOW`, `REVIEW`, or `REJECT`.
9. **Document movement**: Lambda copies the object to the destination, verifies it, and then deletes the source object.
10. **Status retrieval**: The Frontend calls the API to display document status and history.

#### AWS Services Used

+ **Amazon Cognito**: User authentication and `Users`/`Admins` group management.
+ **Amazon API Gateway**: Provides HTTP APIs and protects routes with a JWT Authorizer.
+ **AWS Lambda**: Generates Presigned URLs, processes scan results, performs AI analysis, executes decisions, and retrieves document data.
+ **Amazon S3**: Hosts the Frontend and stores documents in the `quarantine/`, `clean/`, and `reject/` areas.
+ **Amazon CloudFront**: Delivers the Frontend over HTTPS.
+ **AWS WAF**: Protects CloudFront and blocks invalid requests.
+ **Amazon GuardDuty Malware Protection for S3**: Scans newly uploaded objects for malware.
+ **Amazon EventBridge**: Receives malware scan result events.
+ **Amazon SQS**: Decouples components and supports asynchronous processing.
+ **Amazon Bedrock Mantle**: Analyzes document content and evaluates risk.
+ **Amazon DynamoDB**: Stores metadata, processing states, scan results, and final decisions.
+ **AWS Secrets Manager**: Stores the Bedrock Mantle API Key.
+ **AWS KMS**: Encrypts stored data.
+ **Amazon CloudWatch** and **AWS CloudTrail**: Provide monitoring, logging, and auditing.
+ **AWS IAM**: Applies least-privilege permissions to each component.

### 4. Technical Implementation

#### Implementation Phases

1. **Requirements Analysis and Architecture Design**
   + Define user flows, supported file types, size limits, and document states.
   + Design the serverless and event-driven architecture.

2. **Authentication and Storage**
   + Create the Cognito User Pool, App Client, and user groups.
   + Create S3 storage, the DynamoDB table, document prefixes, and CORS rules.

3. **Upload Function**
   + Create the Presigned URL Lambda.
   + Create the `POST /upload-url` route.
   + Test Frontend uploads to `quarantine/`.

4. **Malware Scanning**
   + Enable GuardDuty Malware Protection for S3.
   + Configure EventBridge and SQS.
   + Store scan results in DynamoDB.

5. **AI Content Analysis**
   + Package the `pypdf` and `python-docx` libraries.
   + Store the Mantle API Key in Secrets Manager.
   + Extract content and perform risk analysis.

6. **Decision Engine**
   + Apply GuardDuty priority rules.
   + Process `ALLOW`, `REVIEW`, and `REJECT`.
   + Verify the destination object before deleting the source object.

7. **Frontend Completion and Testing**
   + Display document lists, statuses, and details.
   + Test safe, rejected, and manual-review documents.
   + Deploy the Frontend through CloudFront.

#### Technical Requirements

+ **Primary AWS Region**: `ap-southeast-1`.
+ **Bedrock Mantle Region**: `ap-northeast-1`.
+ **Lambda Runtime**: Python 3.14.
+ **Frontend**: HTML, CSS, and JavaScript.
+ **Initial Supported File Types**: `.txt`, `.pdf`, `.docx`.
+ **Maximum File Size**: 10 MB.
+ **Presigned URL Expiration**: 300 seconds.
+ **S3 Security**: Block Public Access, SSE-KMS encryption, and access through IAM or Presigned URLs.
+ **API Security**: JWT Authorizer and Cognito group validation.
+ **Event Processing**: Idempotent processing to reduce duplicate handling.
+ **Monitoring**: CloudWatch Logs, metrics, alarms, and CloudTrail.

### 5. Roadmap and Milestones

+ **Phase 1 – Analysis and Design**: Complete requirements, architecture diagrams, and the data model.
+ **Phase 2 – Authentication, Storage, and Upload**: Complete Cognito, S3, DynamoDB, API Gateway, and Presigned URL integration.
+ **Phase 3 – Malware Scanning and AI Analysis**: Integrate GuardDuty, EventBridge, SQS, Bedrock Mantle, and Secrets Manager.
+ **Phase 4 – Decision Engine and Frontend**: Complete `ALLOW`, `REVIEW`, and `REJECT` processing, document history, and the detail interface.
+ **Phase 5 – Testing and Deployment**: Perform end-to-end, authorization, CORS, security, and CloudFront deployment testing.
+ **Post-Deployment**: Add secure downloads, search, deletion, notifications, and administrative reporting.

### 6. Budget Estimate

Costs depend on the number of uploaded documents, file sizes, CloudFront traffic, GuardDuty-scanned data volume, and the number of AI tokens processed.

The main cost categories are:

+ Amazon S3 and CloudFront.
+ AWS Lambda and Amazon API Gateway.
+ Amazon DynamoDB.
+ Amazon SQS and Amazon EventBridge.
+ GuardDuty Malware Protection for S3.
+ Amazon Bedrock Mantle.
+ AWS Secrets Manager, AWS KMS, CloudWatch, CloudTrail, and AWS WAF.

For a low-volume demonstration environment, the initial planning budget is:

```text
USD 5–30 per month
```

This range is a planning placeholder and is not an official AWS quotation. The team should update the estimate with AWS Pricing Calculator based on the number of uploaded files, average file size, GuardDuty-scanned volume, AI tokens, API requests, and retention period.

No dedicated hardware cost is required because the platform is fully cloud-based.

### 7. Risk Assessment

#### Risk Matrix

+ **Incorrect AI classification**: High impact, medium probability.
+ **GuardDuty cannot scan a file**: High impact, low-to-medium probability.
+ **Overly broad IAM permissions**: High impact, medium probability.
+ **Presigned URL exposure or expiration**: Medium impact, medium probability.
+ **Duplicate EventBridge or SQS events**: Medium impact, medium probability.
+ **Object copy failure**: High impact, low probability.
+ **AI or malware scanning cost increases**: Medium impact, medium probability.

#### Mitigation Strategies

+ Treat `THREATS_FOUND` as an automatic `REJECT` result.
+ Send ambiguous cases to `MANUAL_REVIEW`.
+ Apply IAM least privilege and use separate roles for Lambda functions.
+ Limit Presigned URLs to 300 seconds.
+ Validate file name, Content-Type, and size before generating a URL.
+ Use DynamoDB conditional updates and event IDs to reduce duplicate processing.
+ Verify the destination ETag and object size before deleting the source object.
+ Store credentials in AWS Secrets Manager.
+ Configure AWS Budgets and CloudWatch Alarms.

#### Contingency Plan

+ Documents with scan or AI errors remain in `quarantine/` and are sent for manual review.
+ Lambda processing can be retried through SQS after temporary failures.
+ The source object is not deleted until the destination object has been verified.
+ DynamoDB stores intermediate states so processing can continue after service recovery.
+ AI analysis can be temporarily disabled while malware scanning remains active if Bedrock Mantle is unavailable.

### 8. Expected Outcomes

#### Technical Improvements

+ Automates document upload, malware scanning, and content analysis.
+ Separates unverified, safe, and rejected documents.
+ Tracks the complete document lifecycle in DynamoDB.
+ Protects APIs with Cognito JWT authentication and User/Admin authorization.
+ Applies serverless and event-driven architecture patterns.
+ Provides logs and processing history for troubleshooting and auditing.

#### Long-Term Value

+ Can be expanded into an internal enterprise document management platform.
+ Can support more file formats and additional AI models.
+ Can integrate notifications through email, Slack, or Microsoft Teams.
+ Can add administrative dashboards, risk analytics, and compliance reports.
+ Can apply S3 lifecycle policies and long-term archival.
+ Provides a practical platform for learning AWS Serverless, Cloud Security, Event-Driven Architecture, and AI.

![Secure AI-Driven Document Platform Architecture Overview](/images/2-Proposal/kientrucaws.png)

#### Video demo
https://drive.google.com/drive/folders/107I3wMFQx0EnR0jGuWC703YX0urXKBPx

#### Link website
https://d2aoitb7o6jdem.cloudfront.net/login