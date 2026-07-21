---
title : "Generate a Download URL"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.8.4 </b> "
---

In this step, we will use the `GenerateDownloadUrlLambda` function to generate an Amazon S3 Presigned URL for a safe document.

The Lambda function is integrated with the following route:

```text
GET /documents/{documentId}/download-url
```

Configure the following environment variables:

```text
DOCUMENTS_TABLE      = SecureDocuments
DOCUMENT_BUCKET      = secure-ai-document-storage
DOWNLOAD_URL_EXPIRES = 300
S3_REGION            = ap-southeast-1
```

The Lambda function retrieves the `documentId`, reads the document metadata from the `SecureDocuments` table, and verifies ownership using the `sub` claim in the JWT token.

A document can be downloaded only if all of the following conditions are satisfied:

```text
status = SAFE
finalDecision = ALLOW
currentPrefix = clean
s3Key starts with clean/
```

If any condition is not met, the API returns **409 Conflict** with one of the following messages:

```text
Document is not in the SAFE state
Document is not allowed to be downloaded
Document is not stored in the clean area
Invalid S3 object key
```

The Amazon S3 client is configured with:

```text
Region: ap-southeast-1
Signature version: s3v4
Addressing style: virtual
```

The Presigned URL is generated using:

```python
s3_client.generate_presigned_url(
    ClientMethod="get_object",
    Params=parameters,
    ExpiresIn=DOWNLOAD_URL_EXPIRES,
    HttpMethod="GET"
)
```

If the item contains a valid `destinationVersionId`, the Lambda function includes the `VersionId` parameter so that the generated URL downloads the correct object version.

The downloaded file name is taken from the `fileName` attribute after removing quotation marks and newline characters.

The Lambda execution role requires the following permissions:

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

Send the following request:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>/download-url
Authorization: Bearer <ACCESS_TOKEN>
```

A successful response is similar to the following:

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
The Amazon S3 bucket remains private. The Presigned URL grants temporary download access to a single object stored in the `clean/` prefix.
{{% /notice %}}