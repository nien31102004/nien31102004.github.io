---
title : "Test AI Document Analysis"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

To test the AI analysis process, upload a safe document to the system using a Presigned URL.

You can use a PDF, TXT, or DOCX document containing normal content such as a project report, technical documentation, or an AWS user guide.

![Upload test document](/images/5-Workshop/5.6/5.6.4/5.6.4.1.png)

After the upload is complete, wait for Amazon GuardDuty to finish scanning the document for malware.

When GuardDuty returns:

```text
NO_THREATS_FOUND
```

The `SecureDocScanOrchestrator` Lambda function is automatically invoked through Amazon SQS.

Open the [Amazon CloudWatch console](https://console.aws.amazon.com/cloudwatch/), choose **Log groups**, and open the following log entry:

```text
message": "Invoking Bedrock Mantle Chat Completions",
"endpoint": "https://bedrock-mantle.ap-northeast-1.api.aws/v1/chat/completions",
"region": "ap-northeast-1",
"model": "google.gemma-3-4b-it",
"documentId": "df461ff5-6daa-4c3b-a2dd-e32515c30359",
"characterCount": 570
```

![Open Lambda logs](/images/5-Workshop/5.6/5.6.4/5.6.4.2.png)

Verify the document extraction log:

```text
Document text extraction completed
```

The log displays information such as:

+ Detected file type.
+ Number of extracted characters.
+ Whether the extracted content was truncated.
+ Number of PDF pages.
+ Number of paragraphs and tables in the DOCX document.
+ Extraction method used.

![Extraction log](/images/5-Workshop/5.6/5.6.4/5.6.4.3.png)

Next, verify the Bedrock Mantle invocation log:

```text
Invoking Bedrock Mantle Chat Completions
```

When the analysis completes successfully, the following log entry is displayed:

```text
AI analysis completed
```

![AI completed log](/images/5-Workshop/5.6/5.6.4/5.6.4.4.png)

Open Amazon DynamoDB and select the following table:

```text
SecureDocuments
```

Locate the record by `documentId` and verify the following attributes:

+ `aiStatus`: `COMPLETED`
+ `aiProvider`: `BEDROCK_MANTLE`
+ `riskScore`
+ `riskLevel`
+ `aiSummary`
+ `aiConfidence`
+ `aiRecommendedAction`
+ `systemRecommendedAction`
+ `detectedFileType`
+ `extractionMetadata`
+ `decisionStatus`

![DynamoDB AI result](/images/5-Workshop/5.6/5.6.4/5.6.4.5.png)

For a safe document, the expected result is:

```json
{
  "aiStatus": "COMPLETED",
  "riskLevel": "LOW",
  "systemRecommendedAction": "ALLOW",
  "decisionStatus": "READY_FOR_EXECUTION"
}
```

After the analysis result is successfully stored, the Lambda function asynchronously invokes:

```text
SecureDocDecisionEngine
```

The Decision Engine then moves the document to the appropriate storage location based on the final decision.

{{% notice note %}}
Documents with a risk score between `0` and `20`, sufficient confidence, and no detected security threats are recommended as `ALLOW`.
{{% /notice %}}

{{% notice note %}}
Documents with ambiguous analysis results, low confidence, or inconsistencies between the AI model and local security rules are assigned the `REVIEW` status for administrator evaluation.
{{% /notice %}}

{{% notice warning %}}
If a `pypdf` or `python-docx` library error occurs, verify that the required libraries are included in the deployment ZIP package. If a TXT file cannot be decoded, save the file using UTF-8 or another encoding supported by the Lambda function.
{{% /notice %}}