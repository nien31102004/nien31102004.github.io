---
title : "Create Amazon S3 Buckets"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

In this step, we will create three Amazon S3 buckets to separate documents based on their processing status.

## Prepare Bucket Names

Amazon S3 bucket names must be globally unique across AWS. Use your AWS Account ID to avoid naming conflicts:

```text
securedocs-quarantine-<account-id>
securedocs-clean-<account-id>
securedocs-rejected-<account-id>
```

## Create the Quarantine Bucket

1. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/).
2. Choose **General purpose buckets**.
3. Choose **Create bucket**.
4. Configure the following settings:

+ **Bucket type**: `General purpose`
+ **Bucket name**: `securedocs-quarantine-<account-id>`
+ **AWS Region**: `Asia Pacific (Singapore) ap-southeast-1`
+ **Object Ownership**: `Bucket owner enforced`
+ **Block Public Access**: Leave all settings enabled.
+ **Bucket Versioning**: `Disable` for this workshop.

![Create quarantine bucket](/images/5-Workshop/5.3/5.3.1/5.3.1.1.png)

5. In the **Default encryption** section, configure:

+ **Encryption type**: `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`
+ **AWS KMS key**: `AWS managed key (aws/s3)`

![Configure S3 encryption](/images/5-Workshop/5.3/5.3.1/5.3.1.2.png)

6. Choose **Create bucket**.

## Create the Clean Bucket

Repeat the same steps and use the following bucket name:

```text
securedocs-clean-<account-id>
```

This bucket stores only documents that have successfully passed malware scanning and AI content analysis.

## Create the Rejected Bucket

Repeat the same steps and use the following bucket name:

```text
securedocs-rejected-<account-id>
```

This bucket stores documents that are identified as containing malware or violating the platform's security policies.

After all three buckets have been created, verify that they appear in the Amazon S3 bucket list.

![S3 bucket list](/images/5-Workshop/5.3/5.3.1/5.3.1.3.png)

{{% notice note %}}
Keep **Block Public Access** enabled for all three buckets. Documents are uploaded through Presigned URLs, so public access to the buckets is not required.
{{% /notice %}}