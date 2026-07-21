---
title: "Week 11 Worklog"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

* Consolidate project progress and review the Serverless, Event-Driven architecture of the Secure AI-Driven Document Platform.
* Complete documentation for the Authentication & API module under my responsibility.
* Standardize the sign-in, JWT authentication, User/Admin authorization, and document API flows.
* Prepare evidence, test cases, and the final demonstration script.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Review the overall architecture; confirm the use of HTTP API and validate Cognito, API Gateway, Lambda, S3, SQS, and DynamoDB names in the diagram. | 29/06/2026 | 29/06/2026 | <https://docs.aws.amazon.com/whitepapers/latest/serverless-multi-tier-architectures-api-gateway-lambda/> |
| 3 | Document Cognito User Pool, SPA App Client, Managed Login, Authorization Code Flow with PKCE, and callback URLs. | 30/06/2026 | 30/06/2026 | <https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-managed-login.html> |
| 4 | Standardize the HTTP API JWT Authorizer; document Issuer, Audience, `Authorization: Bearer`, the `sub` claim, and the `/me` test route. | 01/07/2026 | 01/07/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-jwt-authorizer.html> |
| 5 | Complete the Resource Server, `upload/read/admin` scopes, Cognito `users/admins` groups, and two-layer authorization testing. | 02/07/2026 | 02/07/2026 | <https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-define-resource-servers.html> |
| 6 | Coordinate with Backend on document routes, JWT claim extraction, using `sub` as owner ID, and document ownership checks. | 03/07/2026 | 03/07/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html> |
| 7 and Sun | Organize `401/403/200` evidence, complete the test matrix and demo script, and review the project report. | 04/07/2026 | 05/07/2026 | <https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/> |

### Week 11 Achievements:

* Standardized the project architecture around HTTP API and a Serverless, Event-Driven workflow.
* Completed Authentication & API documentation covering Cognito, OAuth 2.0 with PKCE, JWT Authorizer, scopes, and groups.
* Consolidated test results: no token `401`, fake token `401`, valid token `200`, user accessing admin API `403`, and admin access `200`.
* Aligned with Backend on extracting `sub` from JWT, rejecting frontend-provided user IDs, and enforcing document ownership.
* Completed evidence collection, test cases, and the final demo script.

### End-of-week Evaluation:

* The main objectives of the week were completed as planned.
* The activities strengthened my theoretical knowledge and practical AWS implementation skills.
* Issues encountered during the work were documented and resolved through AWS documentation and discussions with the mentor and team members.
* I reviewed the created resources and consolidated the necessary material for the internship report.
