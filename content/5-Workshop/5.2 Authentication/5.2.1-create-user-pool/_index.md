---
title : "Create Amazon Cognito User Pool"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.2.1 </b> "
---

Amazon Cognito User Pool is used to manage user accounts, authenticate users, and provide JWT Tokens for the application.

1. Open the [Amazon Cognito console](https://console.aws.amazon.com/cognito/).

2. In the Amazon Cognito console, choose **Create user pool**.

3. Under **Application type**, select the application type that matches your web application.

    In the **Sign-in identifiers** section, select:

    **Email**

    Do not select **Username** if users will sign in exclusively with their email addresses.

![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.1.png)

4. Review the configuration and choose **Create user pool**.

    After the User Pool has been created successfully, save the following information:

    - **User Pool ID**
    - **AWS Region**

    These values will be used later when configuring the Frontend application and the JWT Authorizer in Amazon API Gateway.

![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.2.png)

5. Create an App Client by choosing **Create app client**.

![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.3.png)

    After the App Client has been created successfully, review the configuration and save the following information for deployment:

    - **App Client ID**
    - Authentication configuration
    - Hosted UI configuration (if enabled)

    These values will be required when deploying and configuring the Frontend application.

![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.4.png)

6. Edit the **Managed login pages configuration** as follows:

    - Use **http://localhost** during local development.
    - Add the **Amazon CloudFront** URL after deployment so the application can authenticate correctly in the production environment.

![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.5.png)