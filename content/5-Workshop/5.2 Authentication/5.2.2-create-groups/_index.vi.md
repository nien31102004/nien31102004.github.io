---
title : "Tạo nhóm người dùng"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.2.3 </b> "
---

Mở [Amazon Cognito console](https://console.aws.amazon.com/cognito/) và chọn User Pool đã tạo cho dự án.

![Select user pool](/images/5-Workshop/5.2/5.2.2/5.2.2.1.png)

Trong giao diện User Pool, chọn **Groups**, sau đó chọn **Create group**.

Tạo nhóm người dùng thông thường với các thông tin:

+ **Group name**: `Users`
+ **Description**: Nhóm người dùng tải lên, xem và quản lý tài liệu cá nhân.
+ **Precedence**: `10`

Sau đó chọn **Create group**.

![Create group user](/images/5-Workshop/5.2/5.2.2/5.2.2..png)

Tiếp tục chọn **Create group** để tạo nhóm quản trị viên với các thông tin:

+ **Group name**: `Admins`
+ **Description**: Nhóm quản trị viên quản lý người dùng và kiểm duyệt tài liệu.
+ **Precedence**: `1`

Sau đó chọn **Create group**.

![Create group admin](/images/5-Workshop/5.2/5.2.2/5.2.2.2.png)

Kiểm tra danh sách và xác nhận hai nhóm `Users` và `Admins` đã được tạo thành công.

![Group list](/images/5-Workshop/5.2/5.2.2/5.2.2.3.png)

Để thêm tài khoản vào nhóm, chọn tên nhóm, chọn **Add user to group**, chọn tài khoản cần thêm và nhấn **Add**.

![Add user to group](/images/5-Workshop/5.2/5.2.2/5.2.2.5.png)
![List user and group](/images/5-Workshop/5.2/5.2.2/5.2.2.4.png)

{{% notice note %}}
Nhóm `Users` được sử dụng cho người dùng thông thường. Nhóm `Admins` được sử dụng cho các tài khoản có quyền quản lý người dùng và đánh giá tài liệu. Sau khi đăng nhập, thông tin nhóm được lưu trong claim `cognito:groups` của JWT Token.
{{% /notice %}}

Test trang login ở app client sau đó chọn login pages, view login page để test login hoặc postman
![Test login page](/images/5-Workshop/5.2/5.2.2/5.2.2.6.png)