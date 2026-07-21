---
title : "API tài liệu"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8 </b> "
---

Trong phần này, chúng ta sẽ xây dựng nhóm API cho phép người dùng quản lý các tài liệu đã tải lên hệ thống.

Các API được triển khai bằng Amazon API Gateway HTTP API, AWS Lambda, Amazon DynamoDB và Amazon S3. Amazon Cognito JWT Authorizer được sử dụng để xác thực người dùng trước khi chuyển yêu cầu đến Lambda.

API của dự án:

```text
secure-document-api
```

Stage được sử dụng:

```text
$default
```

Bảng DynamoDB lưu metadata tài liệu:

```text
SecureDocuments
```

Bucket Amazon S3 lưu tài liệu:

```text
secure-ai-document-storage
```

Các chức năng trong phần này gồm:

```text
GET    /documents
GET    /documents/{documentId}
DELETE /documents/{documentId}
GET    /documents/{documentId}/download-url
GET    /documents/{documentId}/scan-result
```

Mỗi Lambda lấy định danh người dùng từ claim `sub` trong JWT Token. Trước khi trả dữ liệu, tải tài liệu hoặc xóa tài liệu, Lambda đều kiểm tra thuộc tính `userId` của tài liệu có trùng với người dùng đang đăng nhập hay không.

{{% notice note %}}
Các tài liệu bị đánh dấu `DELETED` không còn được hiển thị trong danh sách tài liệu. Tài liệu chỉ được tải xuống khi đã có trạng thái `SAFE`, quyết định cuối cùng là `ALLOW` và object nằm trong prefix `clean/`.
{{% /notice %}}
