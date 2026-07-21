---
title : "Create Prefixes in Amazon S3"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

In Amazon S3, the folders displayed in the console are actually **prefixes** within the object key. Prefixes help organize documents according to their processing status.

## Create a Prefix for the Quarantine Bucket

1. Open the bucket:

```text
securedocs-quarantine-<account-id>
```

2. Choose **Objects**.
3. Choose **Create folder**.
4. Enter the folder name:

```text
uploads
```

5. Choose **Create folder**.

![Create uploads prefix](/images/5-Workshop/5.3/5.3.3/5.3.3.1.png)

Uploaded files will later use the following object key structure:

```text
uploads/YYYY/MM/DD/documentId/originalName
```

Example:

```text
uploads/2026/07/20/550e8400/report.pdf
```

## Create a Prefix for the Clean Bucket

In the `securedocs-clean-<account-id>` bucket, create the following prefix:

```text
documents
```

![Create documents prefix](/images/5-Workshop/5.3/5.3.3/5.3.3.2.png)

Verified documents will be stored using the following structure:

```text
documents/documentId/originalName
```

## Create a Prefix for the Rejected Bucket

In the `securedocs-rejected-<account-id>` bucket, create the following prefix:

```text
rejected
```

![Create rejected prefix](/images/5-Workshop/5.3/5.3.3/5.3.3.3.png)

Rejected documents will be stored using the following structure:

```text
rejected/documentId/originalName
```

## Environment Variables Used Later

```text
QUARANTINE_BUCKET=securedocs-quarantine-<account-id>
CLEAN_BUCKET=securedocs-clean-<account-id>
REJECTED_BUCKET=securedocs-rejected-<account-id>

QUARANTINE_PREFIX=uploads/
CLEAN_PREFIX=documents/
REJECTED_PREFIX=rejected/
```

{{% notice note %}}
You do not need to manually create the year, month, and day prefixes. When the Lambda function uploads an object using the corresponding key, the Amazon S3 Console automatically displays the prefix hierarchy.
{{% /notice %}}