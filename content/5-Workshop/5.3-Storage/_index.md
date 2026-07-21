---
title : "Storage"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

In this section, we will build the storage layer for the **Secure AI-Driven Document Platform** using Amazon S3 and Amazon DynamoDB.

The following tasks will be completed:

1. [Create Amazon S3 Buckets](5.3.1-create-s3/)
2. [Create an Amazon DynamoDB Table](5.3.2-create-dynamodb/)
3. [Create Prefixes for Document Organization](5.3.3-create-prefix/)
4. [Configure Amazon S3 CORS](5.3.4-configure-cors/)

After completing this section, the system will have three document storage areas:

+ **Quarantine bucket**: Stores newly uploaded documents waiting for malware scanning and AI analysis.
+ **Clean bucket**: Stores documents that have been verified as safe.
+ **Rejected bucket**: Stores documents identified as unsafe or in violation of security policies.

Amazon DynamoDB is used to store document metadata, malware scan status, AI analysis results, and the final decision for each document.