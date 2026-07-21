---
title : "Xác thực người dùng"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

#### Xác thực và phân quyền với Amazon Cognito

Trong phần này, bạn sẽ cấu hình **Amazon Cognito User Pool** để quản lý tài khoản, xác thực người dùng và cung cấp **JWT Token** cho ứng dụng. Bạn cũng sẽ tạo **App Client**, cấu hình trang đăng nhập, tạo các nhóm **Users** và **Admins**, đồng thời kiểm tra luồng đăng nhập trước khi tích hợp với Frontend và Amazon API Gateway.

Thông tin nhóm người dùng được lưu trong claim `cognito:groups` của JWT Token, giúp hệ thống phân biệt quyền của người dùng thông thường và quản trị viên khi truy cập các API.

![overview](/images/5-Workshop/5.2-Authentication/diagram.png)

#### Nội dung

- [Tạo Amazon Cognito User Pool](5.2.1-create-user-pool/)
- [Tạo App Client](5.2.2-create-app-client/)
- [Tạo nhóm người dùng](5.2.3-create-groups/)
- [Kiểm tra đăng nhập](5.2.4-test-login/)