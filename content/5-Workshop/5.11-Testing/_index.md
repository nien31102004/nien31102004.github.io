---
title : "Testing"
date : 2024-01-01
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

This section validates the complete secure document workflow, from authentication and upload to malware scanning, AI analysis, final decision, administration, and CloudFront delivery.

The objective is not only to confirm successful requests. Testing must prove that unsafe documents cannot be downloaded, users cannot access one another's documents, duplicate events do not create duplicate moves, and frontend controls are backed by server-side authorization.

## Prepare the test environment

Create three test accounts in the Cognito user pool:

| Account | Group | Purpose |
|---|---|---|
| `workshop-user-a` | `Users` | Primary user workflow |
| `workshop-user-b` | `Users` | Ownership-isolation tests |
| `workshop-admin` | `Admins` | Review and user-management tests |

Prepare these non-production test files:

- A small ordinary PDF or text file expected to be safe.
- A harmless document containing controlled high-risk or phishing-like wording for AI testing.
- An ambiguous document expected to require manual review.
- The standard EICAR anti-malware test file used in section 5.5. EICAR is a test signature rather than real malware, but use it only in the isolated workshop account.

Record the current resource values:

```text
CloudFront URL
API Gateway invoke URL
Cognito User Pool and App Client IDs
secure-documents table
securedocs-quarantine-<account-id>
securedocs-clean-<account-id>
securedocs-rejected-<account-id>
```

Before testing, confirm that:

- Cognito callback and sign-out URLs contain the CloudFront domain.
- API Gateway JWT authorization and CORS are enabled for all protected routes.
- S3 CORS permits the CloudFront origin to upload with the required headers.
- GuardDuty, DynamoDB Streams, S3 notifications, and Lambda triggers are enabled.
- `SecureDocDecisionEngine` filters `MALWARE_DETECTED`, `AI_ANALYSIS_COMPLETED`, and `ADMIN_DECISION_READY`.
- CloudFront is deployed, its S3 origin uses OAC, and direct access to the frontend bucket is denied.

Use unique file names for every run, for example `T02-safe-20260721.pdf`. Record each returned `documentId`; do not reuse a previous test document when verifying a different branch.

## Expected state transitions

| Scenario | Expected transition |
|---|---|
| Safe document | `PENDING_UPLOAD → MALWARE_CLEAN → AI_ANALYZING → AI_ANALYSIS_COMPLETED → DECISION_PROCESSING → SAFE` |
| Malware detected | `PENDING_UPLOAD → MALWARE_DETECTED → DECISION_PROCESSING → REJECTED` |
| Manual review | `PENDING_UPLOAD → MALWARE_CLEAN → AI_ANALYZING → AI_ANALYSIS_COMPLETED → DECISION_PROCESSING → MANUAL_REVIEW` |
| Admin decision | `MANUAL_REVIEW → ADMIN_DECISION_READY → DECISION_PROCESSING → SAFE or REJECTED` |
| Processing failure | `MALWARE_SCAN_ERROR`, `AI_ERROR`, or `DECISION_ERROR` |

Some intermediate states can change before the frontend displays them. Validate the final state and use CloudWatch Logs or DynamoDB item history fields to confirm the stages that completed.

## Test matrix

| ID | Test | Expected result |
|---|---|---|
| T01 | Authentication and route authorization | Valid login works; missing token is `401`; normal user Admin call is `403` |
| T02 | Safe document flow | Final `SAFE`; object only in clean bucket; download works |
| T03 | Malware flow | Final `REJECTED`; object in rejected bucket; download is denied |
| T04 | AI decision flow | Risk fields are stored and system action produces the expected branch |
| T05 | Manual Admin review | One `ALLOW` and one `REJECT` complete through Decision Lambda |
| T06 | Ownership isolation | Another user receives `404` and cannot download or delete |
| T07 | Delete workflow | S3 object removed; metadata becomes `DELETED`; list hides item |
| T08 | Idempotency and concurrency | Duplicate event is skipped; concurrent Admin decision returns one conflict |
| T09 | Frontend and CloudFront | HTTPS, callback, deep links, CORS, upload, and logout work |
| T10 | Observability and error handling | Logs correlate by IDs and no secrets or unsafe content are exposed |

## T01 — Test authentication and authorization

1. Open the CloudFront URL and sign in as `workshop-user-a`.
2. Confirm that Cognito returns to `/callback` and the document pages load.
3. Sign out and confirm that a protected route starts login again.
4. Call `GET /documents` without an `Authorization` header; expect `401 Unauthorized`.
5. Sign in as `workshop-user-a` and call `GET /admin/documents`; expect `403 Forbidden`.
6. Sign in as `workshop-admin`; confirm that the Admin page and API now work.

Pass when the browser UI and backend produce the same authorization result. The ability to hide a menu alone is not a pass.

## T02 — Test the safe-document path

1. Sign in as `workshop-user-a` and upload the ordinary safe file.
2. Confirm `POST /upload-url` returns `201` and the S3 `PUT` succeeds without a Cognito authorization header.
3. Poll `GET /documents/{documentId}/scan-result` until a terminal state or the workshop timeout is reached.
4. Open the item in `secure-documents`.

Expected DynamoDB fields include:

```json
{
  "status": "SAFE",
  "malwareStatus": "NO_THREATS_FOUND",
  "aiStatus": "COMPLETED",
  "decisionStatus": "COMPLETED",
  "finalDecision": "ALLOW",
  "bucket": "securedocs-clean-<account-id>",
  "currentPrefix": "documents",
  "sourceCleanupStatus": "COMPLETED"
}
```

Confirm that `s3Key` starts with `documents/<documentId>/`, the clean object exists, and the former `quarantineKey` no longer exists in the quarantine bucket. Generate a download URL, download the file, and compare its size with the original.

## T03 — Test malware rejection

Upload the EICAR test file through the normal frontend flow. Do not upload real malware.

Expected result:

```json
{
  "status": "REJECTED",
  "malwareStatus": "THREATS_FOUND",
  "finalDecision": "REJECT",
  "finalDecisionSource": "GUARDDUTY",
  "bucket": "securedocs-rejected-<account-id>",
  "currentPrefix": "rejected"
}
```

Verify that the object exists only under `rejected/<documentId>/`, the quarantine source was cleaned up, and both the details screen and direct call to the download-url endpoint refuse download. An administrator must not be able to change this document to `ALLOW`.

## T04 — Test AI analysis and system decisions

Upload controlled, harmless documents representing low, medium, and high risk. Because generative-model output can vary, record the actual `riskScore`, `riskLevel`, `aiConfidence`, `aiRecommendedAction`, `systemRecommendedAction`, and summary for each run.

Confirm that:

- AI analysis runs only after `malwareStatus = NO_THREATS_FOUND`.
- The item reaches `AI_ANALYSIS_COMPLETED` before the Decision Lambda claims it.
- `ALLOW` ends in `SAFE`, `REJECT` ends in `REJECTED`, and `REVIEW` ends in `MANUAL_REVIEW`.
- A parse or model error becomes `AI_ERROR`; it does not silently release the document.
- The frontend displays returned analysis as text and never renders untrusted extracted content as HTML.

## T05 — Test manual review

Create two clean documents that reach `MANUAL_REVIEW`.

For the first document:

1. Open its Admin review page.
2. Submit `ALLOW` with a reason.
3. Confirm the API returns `202` with `ADMIN_DECISION_READY`.
4. Wait for the Decision Lambda and verify final `SAFE`, `finalDecisionSource = ADMIN_REVIEW`, and the object under `documents/` in the clean bucket.

For the second document, submit `REJECT` and verify final `REJECTED` plus the object under `rejected/` in the rejected bucket.

For both items, confirm `reviewedBy`, `adminReason`, `reviewedAt`, and the final-decision timestamps. The Admin review Lambda must not have direct S3 write or delete permission.

## T06 — Test ownership isolation

Use a `documentId` owned by `workshop-user-a`, then sign in as `workshop-user-b` and call:

```text
GET    /documents/{documentId}
GET    /documents/{documentId}/scan-result
GET    /documents/{documentId}/download-url
DELETE /documents/{documentId}
```

Every request must return `404 Not Found`, and the DynamoDB item and S3 object must remain unchanged. User B's document list must not contain User A's item. Repeat with a random ID and confirm that the response does not reveal whether an ID exists.

## T07 — Test deletion

Delete a terminal-state test document owned by the signed-in user.

Confirm:

- The API returns `204 No Content`.
- DynamoDB contains `status = DELETED`, `deletedAt`, and `updatedAt`.
- The current S3 object is removed from the validated bucket and prefix.
- `GET /documents` no longer displays the item.
- A second delete does not recreate metadata or expose internal details.

Attempt to delete a document while AI or decision processing owns it. Expect `409 Conflict`, and verify that the processing Lambda can still complete normally.

## T08 — Test retries and concurrent decisions

Use the Lambda console test feature to replay a previously completed DynamoDB stream event for the Decision Lambda. Replace sensitive values with workshop test values and do not modify the real completed item.

The conditional claim should return `SKIPPED` or a conditional conflict without copying or deleting another object. Confirm that there is still only one authoritative destination object.

For an item in `MANUAL_REVIEW`, submit `ALLOW` and `REJECT` nearly simultaneously from two Admin sessions. Exactly one request must return `202`; the other must return `409 Conflict`. The final S3 location and DynamoDB decision must match the successful request.

## T09 — Test CloudFront and browser behavior

Verify all of the following through the CloudFront domain:

- HTTP redirects to HTTPS.
- Direct refresh of `/documents/<documentId>` and `/admin/documents` returns the React application.
- Cognito callback and sign-out use exact allowed URLs.
- API preflight and authenticated requests use the CloudFront origin successfully.
- Presigned upload succeeds with the required S3 headers.
- Direct S3 access to frontend objects remains denied.
- A new deployment updates `index.html` and does not return missing hashed assets.
- Browser developer tools show no mixed-content, CORS, CSP, or unhandled JavaScript errors.

## T10 — Verify logs and failure handling

For each test document, correlate:

```text
API Gateway request ID
Lambda request ID
documentId
ownerId
GuardDuty finding or scan result
DynamoDB status timestamps
Source and destination S3 keys
```

Inspect the CloudWatch log groups for upload, malware result, AI analysis, decision, document API, and Admin functions. Logs may contain identifiers and state changes, but must not contain access tokens, AWS credentials, presigned URLs, passwords, complete document contents, or raw untrusted extracted text.

If a test reaches `MALWARE_SCAN_ERROR`, `AI_ERROR`, or `DECISION_ERROR`, confirm that the object is not downloadable, the source object is retained when the destination was not verified, and the item contains a useful but sanitized error field. Restore the failed permission or configuration before retrying in the isolated workshop environment.

## Record the result

Use this template for evidence:

| Test ID | Date/time | Account | Document ID | Expected | Actual | Evidence | Pass/Fail |
|---|---|---|---|---|---|---|---|
| T01 |  |  | N/A |  |  | Screenshot/log |  |
| T02 |  |  |  | `SAFE` |  | DynamoDB/S3/log |  |
| T03 |  |  |  | `REJECTED` |  | GuardDuty/S3/log |  |
| T04 |  |  |  | AI branch |  | DynamoDB/log |  |
| T05 |  |  |  | Admin decision |  | DynamoDB/S3/log |  |
| T06 |  |  |  | `404` |  | API response |  |
| T07 |  |  |  | `DELETED` |  | DynamoDB/S3 |  |
| T08 |  |  |  | One decision |  | API/log |  |
| T09 |  |  | N/A | Browser flow |  | Browser evidence |  |
| T10 |  |  |  | Safe logs/errors |  | CloudWatch |  |

The workshop passes when all mandatory tests succeed, unsafe content is never downloadable, ownership and Admin boundaries hold, each final document has one authoritative location, source cleanup is complete or explicitly pending, and no secret appears in logs or frontend code.
