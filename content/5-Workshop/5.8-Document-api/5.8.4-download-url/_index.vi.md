---
title : "Tạo đường dẫn tải xuống"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.8.4 </b> "
---

Trong bước này, chúng ta sẽ sử dụng Lambda `GenerateDownloadUrlLambda` để tạo Amazon S3 Presigned URL cho tài liệu an toàn.

Lambda được tích hợp với route:

```text
GET /documents/{documentId}/download-url
```

Các biến môi trường:

```text
DOCUMENTS_TABLE      = SecureDocuments
DOCUMENT_BUCKET      = secure-ai-document-storage
DOWNLOAD_URL_EXPIRES = 300
S3_REGION            = ap-southeast-1
```

Lambda lấy `documentId`, đọc metadata trong bảng `SecureDocuments` và kiểm tra quyền sở hữu bằng claim `sub` trong JWT Token.

Tài liệu chỉ được phép tải xuống khi đồng thời đáp ứng các điều kiện:

```text
status = SAFE
finalDecision = ALLOW
currentPrefix = clean
s3Key bắt đầu bằng clean/
```

Nếu không đáp ứng, API trả `409 Conflict` với một trong các thông báo:

```text
Tài liệu chưa ở trạng thái SAFE
Tài liệu chưa được phép tải xuống
Tài liệu không nằm trong vùng clean
S3 key của tài liệu không hợp lệ
```

Amazon S3 client được cấu hình:

```text
Region: ap-southeast-1
Signature version: s3v4
Addressing style: virtual
```

Presigned URL được tạo bằng:

```python
s3_client.generate_presigned_url(
    ClientMethod="get_object",
    Params=parameters,
    ExpiresIn=DOWNLOAD_URL_EXPIRES,
    HttpMethod="GET"
)
```

Nếu item có `destinationVersionId` hợp lệ, Lambda thêm `VersionId` để URL tải đúng phiên bản object.

Tên file tải xuống được lấy từ `fileName` và loại bỏ dấu ngoặc kép cùng ký tự xuống dòng.

Execution role của Lambda được cấp quyền:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadDocumentMetadata",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments"
    },
    {
      "Sid": "DownloadCleanDocuments",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": "arn:aws:s3:::secure-ai-document-storage/clean/*"
    }
  ]
}
```

Gửi yêu cầu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>/download-url
Authorization: Bearer <ACCESS_TOKEN>
```

Response thành công:

```json
{
  "documentId": "e509813c-52b4-4558-ace0-d07c18fbf74d",
  "fileName": "admin.txt",
  "contentType": "text/plain",
  "fileSize": 84,
  "status": "SAFE",
  "downloadUrl": "https://secure-ai-document-storage.s3.ap-southeast-1.amazonaws.com/...",
  "expiresIn": 300
}
```


{{% notice note %}}
Bucket vẫn được giữ ở chế độ private. Presigned URL chỉ cấp quyền tải tạm thời đối với một object cụ thể trong prefix `clean/`.
{{% /notice %}}
