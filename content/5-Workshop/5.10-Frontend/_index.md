---
title : "Frontend"
date : 2024-01-01
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

In this section, you will build a React single-page application for the secure document platform and deploy it through Amazon S3 and Amazon CloudFront.

The frontend provides:

- Amazon Cognito managed login with Authorization Code Grant and PKCE.
- Authenticated upload through `POST /upload-url` followed by a direct presigned `PUT` to the quarantine bucket.
- Document list, details, processing result, download, and delete actions.
- Administrator pages for document review and user management.
- Private static hosting in Amazon S3, delivered over HTTPS by CloudFront with Origin Access Control.

Complete the pages in order:

1. [Create the Vite application](5.10.1-create-vite/)
2. [Integrate authentication and APIs](5.10.2-integrate-api/)
3. [Deploy the build to Amazon S3](5.10.3-deploy-s3/)
4. [Deploy Amazon CloudFront](5.10.4-deploy-cloudfront/)

The browser never receives AWS access keys and never accesses the data buckets directly except through short-lived S3 presigned URLs. All API authorization remains enforced by API Gateway and Lambda.

{{% notice warning %}}
Values whose names begin with `VITE_` are included in the browser bundle. Use them only for public configuration such as IDs, domains, regions, and API URLs. Never put passwords, client secrets, AWS keys, or presigned URLs in frontend environment files.
{{% /notice %}}
