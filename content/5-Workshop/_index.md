---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---
# Secure AI-Driven Document Platform

In this workshop, you will build a serverless platform on AWS that accepts document uploads, scans them for malware, analyzes their content with AI, and releases only documents that satisfy the security policy.

The solution combines synchronous APIs with an event-driven processing pipeline. Users and administrators interact through a React application, while Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon S3, Amazon DynamoDB, Amazon SQS, Amazon EventBridge, Amazon GuardDuty Malware Protection, Amazon Bedrock Mantle, Amazon CloudFront, and Amazon CloudWatch provide the managed backend services.


## What you will build

By the end of the workshop, the platform will provide:

- Cognito sign-in for normal users and administrators.
- JWT-protected user and Admin APIs.
- Direct browser uploads through short-lived S3 presigned URLs.
- Separate quarantine, clean, and rejected document storage areas.
- Automatic malware scanning with GuardDuty Malware Protection for S3.
- AI analysis for phishing, fraud, social engineering, credential theft, and prompt-injection risk.
- A deterministic Decision Engine that applies malware, AI, and Admin-review outcomes.
- User APIs for listing, inspecting, downloading, and deleting documents.
- Admin workflows for review, document management, and user management.
- A React frontend privately hosted in S3 and delivered through CloudFront over HTTPS.
- End-to-end tests, logs, and a complete resource-cleanup procedure.

## Architecture overview

![Secure AI-Driven Document Platform architecture](/images/2-Proposal/kientrucaws.png)

| Layer | AWS services | Responsibility |
|---|---|---|
| User access | CloudFront, Amazon S3, Amazon Cognito | Deliver the web application and authenticate users |
| API | Amazon API Gateway, AWS Lambda | Authorize requests and implement user and Admin operations |
| Document storage | Amazon S3 | Isolate uploaded, approved, and rejected documents |
| Workflow | Amazon SQS, Amazon EventBridge, DynamoDB Streams | Connect processing stages without tightly coupling them |
| Security analysis | GuardDuty Malware Protection, Bedrock Mantle | Detect malware and analyze document-content risk |
| State | Amazon DynamoDB | Store ownership, status, scan results, AI results, and decisions |
| Operations | Amazon CloudWatch | Record logs and support troubleshooting and verification |

## Document-processing flow

1. A user signs in through Amazon Cognito and receives tokens for the web session.
2. The frontend calls the protected upload API with the Cognito access token.
3. The upload Lambda creates a `documentId`, writes the initial DynamoDB item, and returns a short-lived S3 presigned URL.
4. The browser uploads the file directly to the `uploads/` prefix of the quarantine bucket.
5. Amazon S3 sends the upload event to Amazon SQS, and GuardDuty Malware Protection scans the new object.
6. The GuardDuty result reaches the scan-result Lambda through Amazon EventBridge. The Lambda normalizes the result and updates DynamoDB.
7. Only malware-clean documents proceed to text extraction and AI content analysis through Bedrock Mantle.
8. DynamoDB Streams invokes the Decision Engine when a malware, AI, or Admin-review result is ready.
9. A safe document is copied to the clean bucket; a blocked document is copied to the rejected bucket. The source is deleted only after the destination is verified.
10. The frontend reads the current status through the document APIs. A document requiring human judgment remains in `MANUAL_REVIEW` until an authorized administrator submits a decision.

Confirmed malware always results in rejection and cannot be overridden by an AI or Admin decision.

## Workshop roadmap

Complete the sections in order because each section uses resources created earlier.

| Section | You will complete |
|---|---|
| [5.1 Workshop overview](5.1-workshop-overview/) | Review the problem, architecture, objectives, and expected results |
| [5.2 Authentication](5.2-authentication/) | Create the Cognito user pool, app client, groups, and test users |
| [5.3 Storage](5.3-storage/) | Create the S3 buckets, DynamoDB table, prefixes, and CORS configuration |
| [5.4 Upload](5.4-upload/) | Build the presigned-upload Lambda and API route, then test an upload |
| [5.5 Malware scan](5.5-malware-scan/) | Configure SQS, S3 events, GuardDuty, and scan-result processing |
| [5.6 AI analysis](5.6-ai-analysis/) | Prepare extraction libraries, configure Mantle, and analyze safe documents |
| [5.7 Decision](5.7-decision/) | Create the Decision Engine and route files to clean or rejected storage |
| [5.8 Document API](5.8-document-api/) | Add list, detail, delete, download, and scan-result endpoints |
| [5.9 Admin](5.9-admin/) | Add protected document review and user-management operations |
| [5.10 Frontend](5.10-frontend/) | Build the Vite application and deploy it through S3 and CloudFront |
| [5.11 Testing](5.11-testing/) | Validate security, processing branches, retries, and browser behavior |
| [5.12 Cleanup](5.12-cleanup/) | Remove workshop resources safely and verify that nothing billable remains |

## Security principles used throughout the workshop

- The browser never receives AWS access keys or confidential application secrets.
- API Gateway validates JWTs, while Lambda enforces document ownership and Admin authorization on the server side.
- Presigned URLs are short-lived and limited to the intended S3 operation and object key.
- Uploaded content remains quarantined until all required checks are complete.
- Document text is treated as untrusted input; the AI must classify it without following instructions embedded in it.
- DynamoDB conditional updates and idempotent handlers protect the workflow from duplicate events and competing decisions.
- The Decision Engine verifies the destination copy before deleting the quarantine source.
- Logs contain identifiers and status transitions, but must not contain tokens, secrets, full document contents, or presigned URLs.

## Before you begin

Prepare the following:

- An AWS account or workshop sandbox with permission to create the services used in this guide.
- Access to the AWS Management Console and a terminal with the AWS CLI configured when a command-line step is required.
- A local development environment for the Lambda packages and Vite frontend.
- A modern browser for Cognito, API, upload, and CloudFront tests.
- Non-sensitive sample TXT, PDF, and DOCX files. Use only the harmless anti-malware test file specified in the malware-testing section; never use real malware.

Unless a section states otherwise, deploy the main workload in `ap-southeast-1`. Pay close attention to the region shown in each procedure, especially when configuring the Bedrock Mantle credential and global services such as CloudFront and IAM.

Record generated values such as bucket names, API URLs, Cognito IDs, Lambda names, and the CloudFront domain. Later sections reuse these values.

## Completion criteria

The workshop is complete when you can demonstrate that:

- A normal user can sign in, upload a safe document, follow its status, and download it only after the final decision is `SAFE`.
- Malware and high-risk documents are rejected and cannot be downloaded.
- One user cannot view, delete, or download another user's document.
- An administrator can review eligible documents but cannot override confirmed malware.
- Duplicate events and concurrent decisions do not create conflicting metadata or multiple authoritative copies.
- The application works through the CloudFront HTTPS domain while the frontend S3 bucket remains private.
- CloudWatch logs allow each test to be traced without exposing credentials or sensitive content.
- All workshop resources are removed after the final validation.

Select **5.1 Workshop overview** to begin.
