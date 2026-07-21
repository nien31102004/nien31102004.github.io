---
title : "Create API for Presigned URL Generation"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

Amazon API Gateway is used to provide an endpoint for the Frontend to invoke the Lambda function and receive a Presigned URL for uploading documents.

Open the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/) and select the HTTP API created for the project.

In the **APIs** page, choose **Create API** under **HTTP API**.

Enter the following information:

+ **API name**: `secure-document-api`

Then choose **Create API**.

![Create HTTP Api](/images/5-Workshop/5.4/5.4.2/5.4.2.1.png)

![Select API](/images/5-Workshop/5.4/5.4.2/5.4.2.2.png)

In the navigation pane, choose **Routes**, then choose **Create**.

Configure the route:

+ **Method**: `POST`
+ **Resource path**: `/upload-url`

![Create route](/images/5-Workshop/5.4/5.4.2/5.4.2.3.png)

Select the `POST /upload-url` route, then choose **Attach integration**.

In the **Integration** section, select:

+ **Integration type**: `Lambda function`
+ **Lambda function**: `secure-doc-generate-upload-url`
+ **Payload format version**: `2.0`

![Attach Lambda](/images/5-Workshop/5.4/5.4.2/5.4.2.4.png)

In the **Authorization** section, select the JWT Authorizer connected to Amazon Cognito.

For **Authorization**, choose **Manage authorizers**, then choose **Create**.

![Create Authorization](/images/5-Workshop/5.4/5.4.2/5.4.2.5.png)
![Configure authorizer](/images/5-Workshop/5.4/5.4.2/5.4.2.6.png)

In the **CORS** section, allow:

+ **Allowed origins**: `http://localhost:5173`
+ **Allowed methods**: `*`
+ **Allowed headers**: `Authorization`, `Content-Type`

After deploying the Frontend, add the CloudFront domain to the list of **Allowed origins**.

![Configure CORS](/images/5-Workshop/5.4/5.4.2/5.4.2.6.png)

Verify that the route has been successfully integrated with the Lambda function.

{{% notice note %}}
The JWT Authorizer requires the Frontend to send an Access Token or ID Token in the `Authorization` header. Users who are not authenticated will receive an `Unauthorized` response.
{{% /notice %}}