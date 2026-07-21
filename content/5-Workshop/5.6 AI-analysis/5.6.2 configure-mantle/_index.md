---
title : "Configure Amazon Bedrock Mantle"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

Amazon Bedrock Mantle is used to send extracted document content to the AI model and receive the risk assessment result in JSON format.

Prepare the following Bedrock Mantle connection information:

+ **Mantle Region**: `ap-northeast-1`
+ **Mantle Base URL**: `https://bedrock-mantle.ap-northeast-1.api.aws/v1`
+ **Model ID**: The model assigned in Amazon Bedrock Mantle.
+ **API Key**: The authentication key used to invoke the Mantle API.

Open the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/) and choose **Store a new secret**.

In the **Secret type** section, choose **Other type of secret** and enter:

```json
{
  "apiKey": "<BEDROCK_MANTLE_API_KEY>"
}
```

![Create secret](/images/5-Workshop/5.6/5.6.2/5.6.2.1.png)

Specify the secret name:

```text
secure-doc/mantle-api-key
```

Then choose **Store** to create the secret.

Save the secret name or ARN for configuring the Lambda function:

```text
secure-doc/mantle-api-key
```

The Lambda execution role must have permission to read the secret:

```json
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "<MANTLE_SECRET_ARN>"
}
```

![Secret name](/images/5-Workshop/5.6/5.6.2/5.6.2.2.png)

{{% notice warning %}}
Do **not** store the Bedrock Mantle API Key directly in the Lambda source code or as an environment variable. Instead, store the API Key in AWS Secrets Manager to reduce the risk of exposing sensitive credentials.
{{% /notice %}}

{{% notice note %}}
The Lambda function uses the Chat Completions-compatible API at the `/chat/completions` endpoint and expects Amazon Bedrock Mantle to return a single valid JSON object for automated processing.
{{% /notice %}}