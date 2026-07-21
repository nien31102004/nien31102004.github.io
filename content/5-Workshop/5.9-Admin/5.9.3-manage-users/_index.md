---
title : "User Management"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.9.3 </b> "
---

In this step, we will use the `AdminUserManagementLambda` function to manage users in the Amazon Cognito User Pool.

Environment variables:

```text
ADMIN_GROUP_NAME   = Admin
COGNITO_REGION     = ap-southeast-1
DEFAULT_LIST_LIMIT = 20
MAX_LIST_LIMIT     = 60
USER_POOL_ID       = ap-southeast-1_ehExCH8ty
```

The Lambda function is integrated with the following routes:

```text
GET  /admin/users
GET  /admin/users/{username}
POST /admin/users/{username}/disable
POST /admin/users/{username}/enable
```

All routes use a JWT Authorizer.

The Lambda function checks the `cognito:groups` claim and only allows accounts that belong to the `Admin` group.

## List users

Route:

```text
GET /admin/users
```

Send the request:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/users
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Supported query parameters:

```text
limit
nextToken
email
username
enabled
status
```

The default limit is `20`, and the maximum limit is `60`.

Example: Search by email

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/users?email=user@example.com
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Filter enabled users:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/users?enabled=true
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Amazon Cognito `ListUsers` supports only one filter at a time. The Lambda function prioritizes the `email` filter, followed by the `username` filter. The `enabled` and `status` filters are applied after Cognito returns the user data.

Response:

```json
{
  "users": [
    {
      "username": "example-user",
      "sub": "example-sub",
      "email": "user@example.com",
      "emailVerified": true,
      "name": "Example User",
      "enabled": true,
      "status": "CONFIRMED",
      "groups": [
        "Users"
      ],
      "createdAt": "2026-07-18T10:00:00+00:00",
      "updatedAt": "2026-07-18T10:00:00+00:00"
    }
  ],
  "count": 1,
  "nextToken": null,
  "filters": {
    "email": null,
    "username": null,
    "enabled": null,
    "status": null
  }
}
```

## Get user details

Route:

```text
GET /admin/users/{username}
```

The response also includes:

```text
phoneNumber
phoneNumberVerified
mfaOptions
preferredMfa
```

## Disable a user account

Route:

```text
POST /admin/users/{username}/disable
```

The Lambda function calls `AdminDisableUser`, then retrieves the user information again using `AdminGetUser` to verify the account status.

Response:

```json
{
  "username": "example-user",
  "enabled": false,
  "status": "CONFIRMED",
  "message": "User account has been disabled successfully."
}
```

## Enable a user account

Route:

```text
POST /admin/users/{username}/enable
```

The Lambda function calls `AdminEnableUser`.

Response:

```json
{
  "username": "example-user",
  "enabled": true,
  "status": "CONFIRMED",
  "message": "User account has been enabled successfully."
}
```

The Lambda function does not allow administrators to disable or enable the account currently being used to send the request.

Execution role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ManageCognitoUsers",
      "Effect": "Allow",
      "Action": [
        "cognito-idp:ListUsers",
        "cognito-idp:AdminGetUser",
        "cognito-idp:AdminListGroupsForUser",
        "cognito-idp:AdminDisableUser",
        "cognito-idp:AdminEnableUser"
      ],
      "Resource": "arn:aws:cognito-idp:ap-southeast-1:950725740411:userpool/ap-southeast-1_ehExCH8ty"
    }
  ]
}
```

![Manage users successfully](/images/5-Workshop/5.9/5.9.3/5.9.3.1.png)

{{% notice note %}}
The Lambda function currently supports listing users, viewing user details, disabling accounts, and enabling accounts. Creating users, deleting users, and changing user group memberships are not implemented in the current version.
{{% /notice %}}