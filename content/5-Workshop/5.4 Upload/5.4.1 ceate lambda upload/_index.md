---
title : "Create Lambda for Presigned URL Generation"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

AWS Lambda is used to generate a `documentId`, store the initial metadata in Amazon DynamoDB, and generate a Presigned URL that allows the Frontend to upload documents directly to Amazon S3.

Open the [AWS Lambda console](https://console.aws.amazon.com/lambda/) and choose **Create function**.

![Create Lambda](/images/5-Workshop/5.4/5.4.1/5.4.1.1.png)

In the Lambda configuration section, enter:

+ **Function name**: `secure-doc-generate-upload-url`
+ **Runtime**: `Python 3.14`
+ **Architecture**: `x86_64`
+ **Execution role**: Select a role with permissions to access Amazon S3, Amazon DynamoDB, and CloudWatch Logs.

Then choose **Create function**.

![Configure Lambda](/images/5-Workshop/5.4/5.4.1/5.4.1.2.png)

In the **Configuration** tab, choose **Environment variables** and add:

| Key | Value |
| --- | --- |
| `DOCUMENTS_TABLE` | `SecureDocuments` |
| `DOCUMENT_BUCKET` | `secure-ai-document-storage` |
| `MAX_FILE_SIZE` | `10485760` |
| `URL_EXPIRATION` | `300` |

![Environment variables](/images/5-Workshop/5.4/5.4.1/5.4.1.3.png)

In the **Code source** section, replace the contents of the `lambda_function.py` file with the following code:

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
    HTTP API JWT Authorizer stores JWT claims at:
    requestContext.authorizer.jwt.claims
    """

    claims = (
        event.get("requestContext", {})
        .get("authorizer", {})
        .get("jwt", {})
        .get("claims", {})
    )

    user_id = claims.get("sub")

    # Used only during the testing phase before Cognito integration.
    if not user_id:
        user_id = event.get("headers", {}).get("x-user-id")

    if not user_id:
        raise ValueError("Unable to identify the user")

    return str(user_id)


def sanitize_file_name(file_name: str) -> str:
    """
    Remove path information and unsafe characters from the file name.
    Keep letters, numbers, periods, hyphens, and underscores.
    """
    file_name = file_name.replace("\\", "/").split("/")[-1]
    file_name = re.sub(r"[^a-zA-Z0-9._-]", "_", file_name)

    if not file_name:
        raise ValueError("Invalid file name")

    return file_name[:180]


def validate_file(
    file_name: str,
    content_type: str,
    file_size: int,
) -> None:
    extension = os.path.splitext(file_name)[1].lower()

    if extension not in ALLOWED_EXTENSIONS:
        raise ValueError("Only PDF, TXT, and DOCX files are supported")

    if content_type not in ALLOWED_CONTENT_TYPES:
        raise ValueError("Unsupported Content-Type")

    if file_size <= 0:
        raise ValueError("File size must be greater than 0")

    if file_size > MAX_FILE_SIZE:
        raise ValueError(
            f"File exceeds the maximum size of {MAX_FILE_SIZE} bytes"
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

        # Prevent overwriting if the documentId already exists.
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
                "message": "Upload URL created successfully",
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
            {"message": "Unable to create the upload URL"},
        )

    except Exception:
        logger.exception("Unexpected error")
        return response(
            500,
            {"message": "Internal server error"},
        )
```

Choose **Deploy** to update the Lambda source code.

The Lambda execution role must include the following permissions:

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
{{% notice warning %}}
Do not add any leading or trailing spaces to the SecureDocuments value in the Environment variables. Extra spaces may cause Lambda to access an incorrect DynamoDB table.
{{% /notice %}}