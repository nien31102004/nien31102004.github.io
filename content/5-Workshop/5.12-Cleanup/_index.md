---
title : "Cleanup"
date : 2024-01-01
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

Delete the workshop resources after testing to prevent unintended cost and remove test data, credentials, and public endpoints.

{{% notice danger %}}
Cleanup is destructive and usually cannot be undone. Before deleting anything, verify the AWS account ID, every active Region, resource name, and whether the resource is shared with another workload. Never use this checklist unchanged in a production account.
{{% /notice %}}

Follow the order below. Removing IAM roles, buckets, or tables too early can leave services unable to finish processing or make dependent resources harder to delete.

## 1. Record evidence and verify scope

Save any screenshots, test results, configuration exports, or logs required for the workshop report. Do not export access tokens, presigned URLs, passwords, API keys, or document contents.

Confirm the current identity and default Region:

```bash
aws sts get-caller-identity
aws configure get region
```

Review at least:

```text
ap-southeast-1   Main workshop resources
ap-northeast-1   Bedrock Mantle API key used in section 5.6
Global           CloudFront and IAM resources
```

Create an inventory before deletion:

| Service | Workshop resources |
|---|---|
| CloudFront | Frontend distribution and OAC |
| S3 | Quarantine, clean, rejected, and frontend buckets |
| Cognito | User pool, app client, domain, groups, and test users |
| API Gateway | `secure-document-api` |
| Lambda | Upload, malware result, AI, decision, document, and Admin functions; any workshop layer |
| DynamoDB | `secure-documents`, stream, indexes, and optional manual backups |
| GuardDuty | Malware Protection plan for the quarantine bucket |
| SQS | `secure-doc-upload-events-demo` and any dead-letter queue |
| EventBridge | Malware-result rule targeting `secure-doc-process-scan-result` |
| Secrets Manager | `secure-doc/mantle-api-key` |
| IAM | Lambda execution roles, GuardDuty role, and workshop policies |
| CloudWatch | Lambda/API log groups, alarms, and dashboards |

If you tagged resources, use Tag Editor or Resource Groups Tagging API as an additional inventory source. Tags do not cover every resource type, so also check each service console.

## 2. Stop new requests

Finish or abandon all active tests and close the frontend.

1. Disable the CloudFront distribution so users can no longer open the application.
2. Delete `secure-document-api`, or first remove its routes if evidence must be retained briefly.
3. Confirm no workshop client is still uploading directly with an unexpired presigned URL.
4. Wait until active Lambda invocations and asynchronous processing have stopped.

Do not delete DynamoDB or the data buckets while the pipeline is still processing an object.

## 3. Disable event sources and malware scanning

Stop asynchronous producers before deleting their consumers:

1. In each Lambda function, disable or delete the DynamoDB event source mappings for `SecureDocAIAnalysis` and `SecureDocDecisionEngine`.
2. Remove the S3 event notification `secure-doc-upload-created-demo` from the quarantine bucket.
3. In EventBridge, remove the target from the malware-result rule, then delete the rule.
4. [Disable the GuardDuty Malware Protection plan](https://docs.aws.amazon.com/guardduty/latest/ug/disable-malware-s3-protected-bucket.html) for `securedocs-quarantine-<account-id>`.
5. Confirm that `secure-doc-upload-events-demo` and any dead-letter queue no longer receive messages.
6. Purge or delete the workshop queues after confirming that no evidence is still required.

Disabling the bucket protection plan does not require disabling the entire GuardDuty service. If GuardDuty was already enabled for the account or protects other workloads, leave the detector and other protection plans unchanged.

Wait until the protection plan is removed before deleting its GuardDuty IAM role or the quarantine bucket.

## 4. Delete CloudFront safely

[CloudFront requires a distribution to be disabled before deletion](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowToDeleteDistribution.html).

1. Open the CloudFront distribution and choose **Disable**.
2. Wait until the updated distribution status is **Deployed** and it is shown as disabled.
3. Delete the distribution.
4. Delete the workshop Origin Access Control after it is no longer attached to a distribution.
5. Remove the CloudFront statement from the frontend bucket policy.

If you configured a custom domain, remove the Route 53 alias record. Delete an ACM certificate only when no other distribution or service uses it. CloudFront changes are global and can take time to propagate; do not assume deletion completed immediately.

## 5. Delete Lambda and API components

After every trigger and API integration is inactive, delete the workshop functions, including:

```text
secure-doc-generate-upload-url
secure-doc-process-scan-result
SecureDocAIAnalysis
SecureDocDecisionEngine
ListMyDocumentsLambda
GetMyDocumentLambda
DeleteMyDocumentLambda
GenerateDownloadUrlLambda
GetDocumentScanResultLambda
AdminListDocumentsFunction
AdminGetDocumentFunction
AdminReviewDocumentFunction
AdminDeleteDocumentFunction
AdminManageUsersFunction
```

Delete any workshop Lambda layer created for document-extraction dependencies. A function name may differ if you changed it during the workshop, so compare the Lambda console with your inventory rather than relying only on this list.

Leave the CloudWatch log groups temporarily if you still need evidence; log groups are not necessarily deleted with their Lambda functions.

## 6. Revoke AI credentials and delete the secret

In the Amazon Bedrock console in `ap-northeast-1`, revoke or delete the Mantle API key created for the workshop. Revoking the upstream credential first ensures that a copied secret can no longer call the service.

Then delete the Secrets Manager secret:

```text
secure-doc/mantle-api-key
```

Use a recovery window when required by your organization. Force deletion without recovery only for a confirmed workshop-only secret and only when your policy permits it. Check for replicas in other Regions and remove those as well.

## 7. Delete Cognito resources

If the user pool was created only for this workshop:

1. Delete the Cognito managed-login domain.
2. Delete the workshop app client.
3. Delete the `Users` and `Admins` test groups or proceed directly to deleting the user pool.
4. Delete the user pool and all test users.

If the user pool is shared, do not delete it. Remove only the workshop app client, callback and sign-out URLs, workshop domain when applicable, test users, and group memberships created for this project.

Deleting frontend code or CloudFront does not revoke already issued JWTs immediately. Removing the API and app client earlier ensures those tokens cannot continue calling the workshop backend.

## 8. Delete the DynamoDB table

Confirm that Lambda event source mappings are gone. [Disable the DynamoDB stream](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html) if you are retaining the table temporarily; otherwise delete:

```text
secure-documents
```

The table deletion also removes its global secondary indexes. Disable deletion protection first if it was enabled.

Review **Backups** separately. On-demand backups can remain after table deletion and continue incurring storage cost. Delete only workshop backups that are not required for evidence or retention.

## 9. Empty and delete every S3 bucket

Delete these workshop buckets:

```text
securedocs-quarantine-<account-id>
securedocs-clean-<account-id>
securedocs-rejected-<account-id>
securedocs-frontend-<account-id>
```

[Every object version and delete marker must be removed before a bucket can be deleted](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html). For each bucket:

1. Remove remaining event notifications, replication configuration, access points, and workshop bucket policies when present.
2. Choose **Empty** in the S3 console and confirm permanent deletion.
3. When versioning is enabled or suspended, verify that **all object versions and all delete markers** are removed.
4. Delete the empty bucket.

{{% notice warning %}}
`aws s3 rm --recursive` removes current objects but is not sufficient by itself for a versioned bucket; it can leave noncurrent versions and delete markers. A bucket cannot be deleted until every version and delete marker is gone.
{{% /notice %}}

Use a read-only check before deleting a bucket:

```bash
aws s3api list-object-versions \
  --bucket securedocs-quarantine-<account-id>
```

The response must contain no remaining `Versions` or `DeleteMarkers`. Repeat for all four buckets.

## 10. Remove IAM, logs, and supporting resources

After the services that use them are gone:

1. Delete workshop inline policies and detach managed policies from Lambda execution roles.
2. Delete the Lambda execution roles and the GuardDuty Malware Protection role.
3. Delete workshop CloudWatch log groups, alarms, dashboards, and saved queries after exporting required evidence.
4. Delete any workshop-specific EventBridge archive, SQS dead-letter queue, SNS topic, or CloudWatch Logs resource policy.
5. Delete a customer-managed KMS key only if it was created solely for the workshop and no encrypted data or other service uses it. KMS deletion is scheduled and has a waiting period.
6. Remove workshop Route 53 records, custom-domain resources, WAF web ACL associations, and ACM certificates only when they are not shared.

Do not delete account-level CloudTrail, GuardDuty detectors, AWS Config, security services, service-linked roles, or shared networking resources merely because the workshop used them.

## 11. Verify cleanup

Use the service consoles and read-only commands to look for leftovers:

```bash
aws s3api list-buckets
aws lambda list-functions --region ap-southeast-1
aws dynamodb list-tables --region ap-southeast-1
aws apigatewayv2 get-apis --region ap-southeast-1
aws sqs list-queues --region ap-southeast-1
aws secretsmanager list-secrets --region ap-southeast-1
aws cloudfront list-distributions
```

Also verify:

- No Malware Protection plan references the quarantine bucket.
- No Lambda event source mapping references the deleted DynamoDB stream or queue.
- No EventBridge rule targets a deleted Lambda.
- No CloudFront distribution or OAC references the frontend bucket.
- No manual DynamoDB backup, S3 version, delete marker, Secrets Manager replica, or log group remains unintentionally.
- The Cognito managed-login domain and CloudFront URL no longer serve the application.

Billing and Cost Explorer data can be delayed. Review the relevant service charges again after the reporting data updates and confirm that no workshop resource continues generating usage.

## Cleanup checklist

| Resource group | Completed |
|---|---|
| Evidence saved and account/Regions verified | ☐ |
| Frontend and API access stopped | ☐ |
| Lambda triggers and EventBridge rules removed | ☐ |
| GuardDuty Malware Protection plan disabled | ☐ |
| CloudFront distribution and OAC deleted | ☐ |
| Lambda functions, layers, API, and queues deleted | ☐ |
| Mantle API key revoked and secret deleted | ☐ |
| Cognito workshop resources deleted | ☐ |
| DynamoDB table and unwanted backups deleted | ☐ |
| All S3 versions, delete markers, and buckets deleted | ☐ |
| IAM roles, logs, alarms, and optional resources deleted | ☐ |
| Final inventory and billing review completed | ☐ |

Cleanup is complete when the final inventory contains no unintended workshop resource and every retained backup, log, shared security service, certificate, or domain has an explicit owner and retention reason.
