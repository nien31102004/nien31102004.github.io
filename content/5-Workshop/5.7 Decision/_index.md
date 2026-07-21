---
title : "Decision"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

In this section, you will build a **Decision Engine** that combines the malware scan results from Amazon GuardDuty with the AI content analysis results to determine the final status of each document.

The Decision Engine supports three processing outcomes:

+ **ALLOW**: The document is considered safe and is moved from the `quarantine/` prefix to the `clean/` prefix.
+ **REJECT**: The document contains malware or presents a high level of risk and is moved to the `reject/` prefix.
+ **REVIEW**: The document does not meet the conditions for an automatic decision and remains in `quarantine/` for manual administrator review.

During processing, the AWS Lambda function updates the document status in Amazon DynamoDB, copies the object to the destination prefix, verifies the new object, and deletes the source object only after the copy operation has been completed successfully.

The following tasks will be completed:

1. [Create the Decision Engine Lambda](5.7.1-create-decision-lambda/)
2. [Move safe documents](5.7.2-move-safe-file/)
3. [Move rejected documents](5.7.3-move-reject-file/)

After completing this section, the system will be able to automatically classify documents based on security scan results and AI analysis while also allowing administrators to process documents that require manual review.

{{% notice warning %}}
If Amazon GuardDuty returns `THREATS_FOUND`, the document must always be processed as `REJECT`. The AI result or a manual decision must not override a malware detection result.
{{% /notice %}}

{{% notice note %}}
Moving a document in Amazon S3 is performed by copying the object to the destination prefix, verifying the new object, and then deleting the original object from `quarantine/`.
{{% /notice %}}