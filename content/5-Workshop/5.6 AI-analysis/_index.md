---
title : "AI Content Analysis"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

In this section, you will build an AI-powered document content analysis workflow for the **Secure AI-Driven Document Platform**.

After Amazon GuardDuty confirms that a document contains no malware with the `NO_THREATS_FOUND` status, an AWS Lambda function downloads the document from Amazon S3, extracts its text content, and sends the extracted data to Amazon Bedrock Mantle for risk assessment.

The system supports content extraction from the following formats:

+ **TXT**: Decoded directly using Python.
+ **PDF**: Extracted using the `pypdf` library.
+ **DOCX**: Extracted using the `python-docx` library.

The AI analysis result is normalized as JSON and stored in Amazon DynamoDB. The stored information includes the risk score, risk level, summary, confidence, and a recommended action such as `ALLOW`, `REVIEW`, or `REJECT`.

The following tasks will be completed:

1. [Prepare document extraction libraries](5.6.1-prepare-document-libraries/)
2. [Configure Amazon Bedrock Mantle](5.6.2-configure-bedrock-mantle/)
3. [Create the AI content analysis Lambda](5.6.3-create-ai-analysis-lambda/)
4. [Test AI document analysis](5.6.4-test-ai-analysis/)

After completing this section, the system will be able to automatically extract document content, analyze indicators such as phishing, social engineering, credential theft, impersonation, payment fraud, and prompt injection, and then send the result to the Decision Engine for the next processing step.

{{% notice warning %}}
Document content is sent to the AI model only after Amazon GuardDuty returns `NO_THREATS_FOUND`. If GuardDuty returns `THREATS_FOUND`, the document must not be sent to Bedrock Mantle.
{{% /notice %}}

{{% notice note %}}
The Bedrock Mantle API Key must be stored in AWS Secrets Manager. Do not store the API Key directly in the Lambda source code or in environment variables.
{{% /notice %}}