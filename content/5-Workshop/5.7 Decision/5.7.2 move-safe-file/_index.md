---
title : "Move Safe Documents"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2 </b> "
---

When the final decision is `ALLOW`, the Decision Engine moves the document from the `quarantine/` prefix to the `clean/` prefix.

A document may be moved to the safe area only when:

+ `malwareStatus` is `NO_THREATS_FOUND`.
+ `aiStatus` is `COMPLETED`.
+ `systemRecommendedAction` is `ALLOW`, or an administrator approves a document that is awaiting review.

The document path is changed as follows:

```text
quarantine/{userId}/{documentId}/{fileName}
```

is changed to:

```text
clean/{userId}/{documentId}/{fileName}
```

The Decision Engine temporarily updates the document status to:

```text
MOVING_TO_CLEAN
```

![Moving to clean](/images/5-Workshop/5.7/5.7.2/5.7.2.1.png)

The Lambda function uses `CopyObject` to copy the document to the `clean/` prefix.

After copying the object, the Lambda function verifies that:

+ The destination object exists.
+ The destination object has a valid ETag.
+ The destination object size matches the source object size.
+ The Version ID is stored when S3 Versioning is enabled.

After successful verification, the DynamoDB record is updated:

+ `status`: `SAFE`
+ `finalDecision`: `ALLOW`
+ `currentPrefix`: `clean`
+ `s3Key`: the new path under `clean/`
+ `finalDecisionSource`: `SYSTEM_RULES` or `ADMIN_REVIEW`
+ `sourceCleanupStatus`: `PENDING`

![Safe document record](/images/5-Workshop/5.7/5.7.2/5.7.2.2.png)

The Lambda function then deletes the original object from the `quarantine/` prefix and updates:

```text
sourceCleanupStatus = COMPLETED
```

![Clean object](/images/5-Workshop/5.7/5.7.2/5.7.2.3.png)

{{% notice note %}}
Moving an object in Amazon S3 consists of two operations: copying the object to the new path and deleting it from the original path.
{{% /notice %}}

{{% notice warning %}}
The Decision Engine verifies the destination object before deleting the source object. This mechanism reduces the risk of data loss if the copy operation fails.
{{% /notice %}}