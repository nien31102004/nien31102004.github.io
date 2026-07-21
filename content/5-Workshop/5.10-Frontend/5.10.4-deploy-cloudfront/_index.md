---
title : "Deploy the Frontend to Amazon S3"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.10.4 </b> "
---

Build the Vite application and upload the generated static files to a dedicated private Amazon S3 bucket for the frontend. Do not reuse the quarantine, clean, or rejected buckets to store website assets.

## Create the frontend bucket

Open the [Amazon S3 console](https://console.aws.amazon.com/s3/) and create the following bucket:

```text
securedocs-frontend-<account-id>
```

It is recommended to use the same AWS Region as the rest of the workshop. Keep the following settings:

- **Object Ownership:** Bucket owner enforced.
- **Block Public Access:** Enable all four options.
- **Bucket Versioning:** Enable if you want to simplify rollback to previous deployments.
- **Default encryption:** SSE-S3 is suitable for public application assets stored in a private bucket. Apply your organization's encryption policy if different requirements exist.

![structure systems](/images/5-Workshop/5.10/5.10.3/5.10.3.1.png)

Do not enable S3 Static Website Hosting. In the next section, the bucket will be used as a standard S3 REST origin with CloudFront Origin Access Control (OAC), allowing the bucket to remain private.

## Create the production environment

Create a `.env.production` file with the following public production configuration:

```text
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=<user-pool-id>
VITE_COGNITO_CLIENT_ID=<app-client-id>
VITE_COGNITO_DOMAIN=<cognito-domain>.auth.ap-southeast-1.amazoncognito.com
VITE_API_BASE_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com
```

The API URL does not include a stage suffix when the HTTP API uses the `$default` stage. Verify that the Cognito App Client does not have a client secret because a browser-based SPA cannot securely protect client secrets.

![evn config](/images/5-Workshop/5.10/5.10.3/5.10.3.1.png)

Build and preview the application:

```bash
npm run build
npm run preview
```

Open the preview URL, verify the main application routes, and confirm that Vite has generated `dist/index.html` together with hashed assets under `dist/assets/`.

## Upload the build to Amazon S3

Use AWS CloudShell or an AWS CLI profile with deployment permissions:

```bash
aws s3 sync dist s3://securedocs-frontend-<account-id> \
  --delete \
  --exclude "index.html" \
  --cache-control "public,max-age=31536000,immutable"

aws s3 cp dist/index.html \
  s3://securedocs-frontend-<account-id>/index.html \
  --content-type "text/html" \
  --cache-control "no-cache,no-store,must-revalidate"
```

![uploads frontend](/images/5-Workshop/5.10/5.10.3/5.10.3.3.png)

The deployment identity only requires the following bucket policy:

```text
{
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::secure-doc-frontend/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::acount_id:distribution/E21XV83QYZOOMQ"
                }
            }
        }
    ]
}
```

## Verify the uploaded build

In the Amazon S3 console, verify that the bucket contains `index.html` and the `assets/` directory. Check the object metadata:

- `index.html` uses `Content-Type: text/html` together with a no-cache policy.
- JavaScript and CSS files have the correct content types and long-term cache-control headers.
- Block Public Access remains enabled.

![uploaded](/images/5-Workshop/5.10/5.10.3/5.10.3.4.png)

Opening the S3 object URL directly should return **Access Denied**. This is the expected behavior because only the CloudFront distribution configured in the next section is allowed to access the bucket.

{{% notice warning %}}
Do not make the frontend bucket public and do not add a public-read ACL or a public `s3:GetObject` bucket policy. CloudFront Origin Access Control (OAC) will provide secure access to the private origin.
{{% /notice %}}