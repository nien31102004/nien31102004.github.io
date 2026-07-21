---
title : "Tạo Lambda cấp Presigned URL"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

AWS Lambda được sử dụng để tạo `documentId`, lưu metadata ban đầu vào Amazon DynamoDB và cấp Presigned URL cho phép Frontend tải tài liệu trực tiếp lên Amazon S3.

Mở [AWS Lambda console](https://console.aws.amazon.com/lambda/) và chọn **Create function**.

![Create Lambda](/images/5-Workshop/5.4/5.4.1/5.4.1.1.png)

Trong phần cấu hình Lambda, nhập:

+ **Function name**: `secure-doc-generate-upload-url`
+ **Runtime**: `Python 3.14`
+ **Architecture**: `x86_64`
+ **Execution role**: Chọn role có quyền truy cập Amazon S3, Amazon DynamoDB và CloudWatch Logs.

Sau đó chọn **Create function**.

![Configure Lambda](/images/5-Workshop/5.4/5.4.1/5.4.1.2.png)

Trong tab **Configuration**, chọn **Environment variables** và thêm:

| Key | Value |
| --- | --- |
| `DOCUMENTS_TABLE` | `SecureDocuments` |
| `DOCUMENT_BUCKET` | `secure-ai-document-storage` |
| `MAX_FILE_SIZE` | `10485760` |
| `URL_EXPIRATION` | `300` |

![Environment variables](/images/5-Workshop/5.4/5.4.1/5.4.1.3.png)

Trong phần **Code source**, thay nội dung file `lambda_function.py` bằng đoạn mã sau:

```python
import base64
import json
import logging
import mimetypes
import os
import re
import uuid
from datetime import datetime, timezone
from decimal import Decimal
from typing import Any

import boto3
from botocore.exceptions import ClientError

logger = logging.getLogger()
logger.setLevel(logging.INFO)

s3_client = boto3.client("s3")
dynamodb = boto3.resource("dynamodb")

TABLE_NAME = os.environ["DOCUMENTS_TABLE"]
BUCKET_NAME = os.environ["DOCUMENTS_BUCKET"]
UPLOAD_URL_EXPIRES = int(os.environ.get("UPLOAD_URL_EXPIRES", "300"))
MAX_FILE_SIZE = int(os.environ.get("MAX_FILE_SIZE", "10485760"))

table = dynamodb.Table(TABLE_NAME)

ALLOWED_CONTENT_TYPES = {
    "application/pdf",
    "text/plain",
    "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
}

ALLOWED_EXTENSIONS = {
    ".pdf",
    ".txt",
    ".docx",
}


def response(status_code: int, body: dict[str, Any]) -> dict[str, Any]:
    return {
        "statusCode": status_code,
        "headers": {
            "content-type": "application/json",
        },
        "body": json.dumps(body, ensure_ascii=False),
    }


def parse_body(event: dict[str, Any]) -> dict[str, Any]:
    raw_body = event.get("body")

    if not raw_body:
        return {}

    if event.get("isBase64Encoded"):
        raw_body = base64.b64decode(raw_body).decode("utf-8")

    return json.loads(raw_body)


def get_user_id(event: dict[str, Any]) -> str:
    """
    HTTP API JWT Authorizer đặt JWT claims tại:
    requestContext.authorizer.jwt.claims
    """

    claims = (
        event.get("requestContext", {})
        .get("authorizer", {})
        .get("jwt", {})
        .get("claims", {})
    )

    user_id = claims.get("sub")

    # Chỉ dùng cho giai đoạn test chưa nối Cognito.
    if not user_id:
        user_id = event.get("headers", {}).get("x-user-id")

    if not user_id:
        raise ValueError("Không xác định được người dùng")

    return str(user_id)


def sanitize_file_name(file_name: str) -> str:
    """
    Loại bỏ đường dẫn và ký tự nguy hiểm trong tên file.
    Giữ chữ, số, dấu chấm, gạch ngang và gạch dưới.
    """
    file_name = file_name.replace("\\", "/").split("/")[-1]
    file_name = re.sub(r"[^a-zA-Z0-9._-]", "_", file_name)

    if not file_name:
        raise ValueError("Tên file không hợp lệ")

    return file_name[:180]


def validate_file(
    file_name: str,
    content_type: str,
    file_size: int,
) -> None:
    extension = os.path.splitext(file_name)[1].lower()

    if extension not in ALLOWED_EXTENSIONS:
        raise ValueError("Chỉ hỗ trợ file PDF, TXT và DOCX")

    if content_type not in ALLOWED_CONTENT_TYPES:
        raise ValueError("Content-Type không được hỗ trợ")

    if file_size <= 0:
        raise ValueError("Kích thước file phải lớn hơn 0")

    if file_size > MAX_FILE_SIZE:
        raise ValueError(
            f"File vượt quá giới hạn {MAX_FILE_SIZE} byte"
        )


def lambda_handler(
    event: dict[str, Any],
    context: Any,
) -> dict[str, Any]:
    try:
        body = parse_body(event)

        original_file_name = sanitize_file_name(
            str(body.get("fileName", "")).strip()
        )
        content_type = str(
            body.get("contentType", "")
        ).strip().lower()
        file_size = int(body.get("fileSize", 0))

        validate_file(
            original_file_name,
            content_type,
            file_size,
        )

        user_id = get_user_id(event)
        document_id = str(uuid.uuid4())
        now = datetime.now(timezone.utc).isoformat()

        s3_key = (
            f"quarantine/{user_id}/"
            f"{document_id}/{original_file_name}"
        )

        item = {
            "documentId": document_id,
            "userId": user_id,
            "originalFileName": original_file_name,
            "contentType": content_type,
            "expectedFileSize": Decimal(str(file_size)),
            "bucket": BUCKET_NAME,
            "s3Key": s3_key,
            "currentPrefix": "quarantine",
            "status": "PENDING_UPLOAD",
            "malwareStatus": "PENDING",
            "aiStatus": "PENDING",
            "riskScore": Decimal("0"),
            "reason": "",
            "createdAt": now,
            "updatedAt": now,
        }

        # Tránh ghi đè nếu documentId vô tình đã tồn tại.
        table.put_item(
            Item=item,
            ConditionExpression="attribute_not_exists(documentId)",
        )

        upload_url = s3_client.generate_presigned_url(
            ClientMethod="put_object",
            Params={
                "Bucket": BUCKET_NAME,
                "Key": s3_key,
                "ContentType": content_type,
                "Metadata": {
                    "document-id": document_id,
                    "user-id": user_id,
                },
            },
            ExpiresIn=UPLOAD_URL_EXPIRES,
            HttpMethod="PUT",
        )

        return response(
            201,
            {
                "message": "Đã tạo URL upload",
                "documentId": document_id,
                "uploadUrl": upload_url,
                "method": "PUT",
                "expiresIn": UPLOAD_URL_EXPIRES,
                "requiredHeaders": {
                    "Content-Type": content_type,
                    "x-amz-meta-document-id": document_id,
                    "x-amz-meta-user-id": user_id,
                },
            },
        )

    except (ValueError, TypeError, json.JSONDecodeError) as exc:
        logger.warning("Invalid request: %s", exc)
        return response(400, {"message": str(exc)})

    except ClientError:
        logger.exception("AWS service error")
        return response(
            500,
            {"message": "Không thể tạo URL upload"},
        )

    except Exception:
        logger.exception("Unexpected error")
        return response(
            500,
            {"message": "Lỗi hệ thống"},
        )
```

Chọn **Deploy** để cập nhật mã nguồn Lambda.

Role thực thi của Lambda cần có các quyền chính:
````Role
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DocumentBucketAccess",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::secure-ai-document-storage/quarantine/*",
                "arn:aws:s3:::secure-ai-document-storage/clean/*",
                "arn:aws:s3:::secure-ai-document-storage/reject/*"
            ]
        },
        {
            "Sid": "DocumentTableAccess",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:GetItem",
                "dynamodb:DeleteItem",
                "dynamodb:Query",
                "dynamodb:UpdateItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments",
                "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments/index/*"
            ]
        }
    ]
}
````
{{% notice warning %}}
Không thêm khoảng trắng ở đầu hoặc cuối giá trị `SecureDocuments` trong Environment variables. Khoảng trắng có thể khiến Lambda truy cập sai tên bảng DynamoDB.
{{% /notice %}}---
