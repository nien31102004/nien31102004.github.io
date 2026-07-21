---
title : "Move Rejected Documents"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3 </b> "
---

When malware or high-risk content is detected, the Decision Engine moves the document from the `quarantine/` prefix to the `reject/` prefix.

A document is rejected when:

+ GuardDuty returns `THREATS_FOUND`.
+ The AI system recommends `REJECT` and the risk validation rules are satisfied.
+ An administrator rejects a document that is awaiting review.

The document path is changed from:

```text
quarantine/{userId}/{documentId}/{fileName}
```

to:

```text
reject/{userId}/{documentId}/{fileName}
```

The Decision Engine temporarily updates the status to:

```text
MOVING_TO_REJECT
```

The Lambda function copies the document to the `reject/` prefix and verifies the destination object before updating the final status.

After processing is completed, DynamoDB is updated:

+ `status`: `REJECTED`
+ `finalDecision`: `REJECT`
+ `currentPrefix`: `reject`
+ `s3Key`: the new path under `reject/`
+ `finalRiskScore`: the final risk score
+ `finalRiskLevel`: the final risk level
+ `finalDecisionSource`: the source of the final decision

![Rejected document record](/images/5-Workshop/5.7/5.7.3/5.7.3.1.png)

The source object in the `quarantine/` prefix is deleted after the destination object has been verified.


If the system result is `REVIEW`, the document is not moved and remains under the `quarantine/` prefix with the following states:

```text
status = MANUAL_REVIEW
reviewStatus = PENDING
finalDecision = PENDING
```

{{% notice warning %}}
If GuardDuty detects `THREATS_FOUND`, the AI result must not override the decision. The document is always processed as `REJECT`.
{{% /notice %}}

{{% notice note %}}
A document with the `MANUAL_REVIEW` status remains in `quarantine/` until an administrator selects `ALLOW` or `REJECT`.
{{% /notice %}}