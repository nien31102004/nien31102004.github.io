---
title : "Create User Groups"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

Open the [Amazon Cognito console](https://console.aws.amazon.com/cognito/) and select the User Pool created for the project.

![Select user pool](/images/5-Workshop/5.2/5.2.2/5.2.2.1.png)

In the User Pool interface, choose **Groups**, then select **Create group**.

Create a regular user group with the following information:

+ **Group name**: `Users`
+ **Description**: Group for users who can upload, view, and manage their own documents.
+ **Precedence**: `10`

Then choose **Create group**.

![Create group user](/images/5-Workshop/5.2/5.2.2/5.2.2.1.png)

Next, choose **Create group** again to create the administrator group with the following information:

+ **Group name**: `Admins`
+ **Description**: Group for administrators who manage users and moderate documents.
+ **Precedence**: `1`

Then choose **Create group**.

![Create group admin](/images/5-Workshop/5.2/5.2.2/5.2.2.2.png)

Verify that both groups, `Users` and `Admins`, have been created successfully.

![Group list](/images/5-Workshop/5.2/5.2.2/5.2.2.3.png)

To add a user to a group, select the group name, choose **Add user to group**, select the user account, and then choose **Add**.

![Add user to group](/images/5-Workshop/5.2/5.2.2/5.2.2.5.png)
![List user and group](/images/5-Workshop/5.2/5.2.2/5.2.2.4.png)

{{% notice note %}}
The `Users` group is intended for regular users. The `Admins` group is used for accounts with permissions to manage users and review documents. After a user signs in, the group membership is included in the `cognito:groups` claim of the JWT Token.
{{% /notice %}}

Test the login page from the App Client by choosing **Login pages** and then **View login page**. You can also test the authentication flow using Postman.

![Test login page](/images/5-Workshop/5.2/5.2.2/5.2.2.6.png)