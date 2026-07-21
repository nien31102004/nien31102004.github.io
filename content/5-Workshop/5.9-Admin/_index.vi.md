---
title : "Quản trị hệ thống"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9 </b> "
---

Trong phần này, chúng ta sẽ xây dựng các API dành cho quản trị viên để quản lý tài liệu, xem trước nội dung cần đánh giá, ghi nhận quyết định kiểm duyệt và quản lý tài khoản người dùng trong Amazon Cognito.

Các API được triển khai bằng Amazon API Gateway HTTP API và AWS Lambda. Amazon Cognito JWT Authorizer được sử dụng để xác thực người gọi trước khi chuyển request đến Lambda.

API của dự án:

```text
secure-document-api
```

Stage:

```text
$default
```

Nhóm quản trị viên được sử dụng trong dự án:

```text
Admin
```

Các chức năng chính:

```text
GET  /admin/documents
GET  /admin/documents/{documentId}/preview-url
GET  /admin/documents/{documentId}/review
POST /admin/documents/{documentId}/review

GET  /admin/users
GET  /admin/users/{username}
POST /admin/users/{username}/disable
POST /admin/users/{username}/enable
```

Quy trình quản trị tài liệu:

```text
Lọc danh sách tài liệu
→ Xem nội dung cần đánh giá
→ Kiểm tra kết quả quét và phân tích AI
→ Gửi quyết định APPROVE hoặc REJECT
→ Gọi SecureDocDecisionEngine
→ Di chuyển tài liệu đến vùng lưu trữ phù hợp
```

{{% notice note %}}
Các Lambda quản trị đều kiểm tra claim `cognito:groups` trong JWT Token. Tài khoản không thuộc nhóm `Admin` sẽ bị từ chối truy cập.
{{% /notice %}}