---
title : "Tạo Prefix trong Amazon S3"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3.3 </b> "
---

Trong Amazon S3, các thư mục hiển thị trên Console thực chất là các **prefix** trong object key. Prefix giúp tổ chức tài liệu rõ ràng theo từng trạng thái xử lý.

## Tạo prefix cho Quarantine bucket

1. Mở bucket:

```text
securedocs-quarantine-<account-id>
```

2. Chọn **Objects**.
3. Chọn **Create folder**.
4. Nhập tên:

```text
uploads
```

5. Chọn **Create folder**.

![Create uploads prefix](/images/5-Workshop/5.3/5.3.3/5.3.3.1.png)

File tải lên sau này có object key dạng:

```text
uploads/YYYY/MM/DD/documentId/originalName
```

Ví dụ:

```text
uploads/2026/07/20/550e8400/report.pdf
```

## Tạo prefix cho Clean bucket

Trong bucket `securedocs-clean-<account-id>`, tạo prefix:

```text
documents
```

![Create documents prefix](/images/5-Workshop/5.3/5.3.3/5.3.3.2.png)

Tài liệu an toàn sẽ được lưu theo cấu trúc:

```text
documents/documentId/originalName
```

## Tạo prefix cho Rejected bucket

Trong bucket `securedocs-rejected-<account-id>`, tạo prefix:

```text
rejected
```

![Create rejected prefix](/images/5-Workshop/5.3/5.3.3/5.3.3.3.png)

Tài liệu bị từ chối sẽ được lưu theo cấu trúc:

```text
rejected/documentId/originalName
```

## Biến môi trường sử dụng sau này

```text
QUARANTINE_BUCKET=securedocs-quarantine-<account-id>
CLEAN_BUCKET=securedocs-clean-<account-id>
REJECTED_BUCKET=securedocs-rejected-<account-id>

QUARANTINE_PREFIX=uploads/
CLEAN_PREFIX=documents/
REJECTED_PREFIX=rejected/
```

{{% notice note %}}
Không cần tạo thủ công các prefix năm, tháng và ngày. Khi Lambda tải object với key tương ứng, Amazon S3 Console sẽ tự hiển thị cấu trúc prefix.
{{% /notice %}}
