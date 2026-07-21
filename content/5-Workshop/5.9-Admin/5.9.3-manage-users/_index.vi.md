---
title : "Quản lý người dùng"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.9.3 </b> "
---

Trong bước này, chúng ta sẽ sử dụng Lambda `AdminUserManagementLambda` để quản lý người dùng trong Amazon Cognito User Pool.

Các biến môi trường:

```text
ADMIN_GROUP_NAME   = Admin
COGNITO_REGION     = ap-southeast-1
DEFAULT_LIST_LIMIT = 20
MAX_LIST_LIMIT     = 60
USER_POOL_ID       = ap-southeast-1_ehExCH8ty
```

Lambda được tích hợp với các route:

```text
GET  /admin/users
GET  /admin/users/{username}
POST /admin/users/{username}/disable
POST /admin/users/{username}/enable
```

Tất cả route đều sử dụng JWT Authorizer.

Lambda kiểm tra claim `cognito:groups` và chỉ cho phép tài khoản thuộc group `Admin`.

## Liệt kê người dùng

Route:

```text
GET /admin/users
```

Gửi yêu cầu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/users
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Các query parameter:

```text
limit
nextToken
email
username
enabled
status
```

Giới hạn mặc định là `20`, tối đa là `60`.

Ví dụ tìm theo email:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/users?email=user@example.com
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Lọc tài khoản đang hoạt động:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/admin/users?enabled=true
Authorization: Bearer <ADMIN_ACCESS_TOKEN>
```

Amazon Cognito `ListUsers` chỉ hỗ trợ một filter tại một thời điểm. Lambda ưu tiên `email`, sau đó mới đến `username`. Các bộ lọc `enabled` và `status` được áp dụng sau khi Cognito trả dữ liệu.

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

## Xem chi tiết người dùng

Route:

```text
GET /admin/users/{username}
```

Response có thêm:

```text
phoneNumber
phoneNumberVerified
mfaOptions
preferredMfa
```

## Khóa tài khoản

Route:

```text
POST /admin/users/{username}/disable
```

Lambda gọi `AdminDisableUser`, sau đó đọc lại thông tin bằng `AdminGetUser` để xác nhận trạng thái.

Response:

```json
{
  "username": "example-user",
  "enabled": false,
  "status": "CONFIRMED",
  "message": "Đã khóa tài khoản thành công"
}
```

## Mở khóa tài khoản

Route:

```text
POST /admin/users/{username}/enable
```

Lambda gọi `AdminEnableUser`.

Response:

```json
{
  "username": "example-user",
  "enabled": true,
  "status": "CONFIRMED",
  "message": "Đã mở khóa tài khoản thành công"
}
```

Lambda không cho phép quản trị viên tự khóa hoặc tự mở khóa tài khoản đang dùng để gửi request.

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
Lambda hiện hỗ trợ liệt kê, xem chi tiết, khóa và mở khóa tài khoản. Mã nguồn không triển khai chức năng tạo, xóa người dùng hoặc thay đổi group.
{{% /notice %}}
