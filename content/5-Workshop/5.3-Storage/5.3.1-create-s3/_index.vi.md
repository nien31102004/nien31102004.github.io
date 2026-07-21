---
title : "Tạo Amazon S3 Bucket"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

Trong bước này, chúng ta sẽ tạo ba Amazon S3 bucket để phân tách tài liệu theo trạng thái xử lý.

## Chuẩn bị tên bucket

Tên S3 bucket phải duy nhất trên toàn AWS. Sử dụng AWS Account ID để tránh trùng tên:

```text
securedocs-quarantine-<account-id>
securedocs-clean-<account-id>
securedocs-rejected-<account-id>
```

## Tạo Quarantine bucket

1. Mở [Amazon S3 console](https://console.aws.amazon.com/s3/).
2. Chọn **General purpose buckets**.
3. Chọn **Create bucket**.
4. Cấu hình các thông tin:

+ **Bucket type**: `General purpose`
+ **Bucket name**: `securedocs-quarantine-<account-id>`
+ **AWS Region**: `Asia Pacific (Singapore) ap-southeast-1`
+ **Object Ownership**: `Bucket owner enforced`
+ **Block Public Access**: giữ bật toàn bộ
+ **Bucket Versioning**: `Disable` cho bản workshop

![Create quarantine bucket](/images/5-Workshop/5.3/5.3.1/5.3.1.1.png)

5. Trong phần **Default encryption**, chọn:

+ **Encryption type**: `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`
+ **AWS KMS key**: `AWS managed key (aws/s3)`

![Configure S3 encryption](/images/5-Workshop/5.3/5.3.1/5.3.1.2.png)

6. Chọn **Create bucket**.

## Tạo Clean bucket

Thực hiện tương tự và đặt tên:

```text
securedocs-clean-<account-id>
```

Bucket này chỉ lưu những tài liệu đã vượt qua bước quét malware và phân tích AI.

## Tạo Rejected bucket

Thực hiện tương tự và đặt tên:

```text
securedocs-rejected-<account-id>
```

Bucket này lưu những tài liệu bị phát hiện có malware hoặc không đáp ứng chính sách của hệ thống.

Sau khi tạo xong, kiểm tra danh sách có đủ ba bucket.

![S3 bucket list](/images/5-Workshop/5.3/5.3.1/5.3.1.3.png)

{{% notice note %}}
Giữ **Block Public Access** ở trạng thái bật cho cả ba bucket. Frontend sẽ tải tài liệu lên bằng Presigned URL nên không cần mở quyền public cho bucket.
{{% /notice %}}
