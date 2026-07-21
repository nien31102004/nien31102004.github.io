---
title : "Tạo Amazon Cognito User Pool"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.2.1 </b> "
---

Amazon Cognito User Pool được sử dụng để quản lý tài khoản, xác thực người dùng và cung cấp JWT Token cho ứng dụng.
1. Mở [Amazon Cognito console](https://console.aws.amazon.com/cognito/).
2. Trong giao diện Amazon Cognito, chọn **Create user pool**.
3. Trong phần **Application type**, chọn loại ứng dụng phù hợp với ứng dụng web của dự án.
    Trong phần Sign-in identifiers, chọn: Email Không chọn thêm Username nếu người dùng đăng nhập hoàn toàn bằng địa chỉ email.
![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.1.png)
4. Kiểm tra lại cấu hình và chọn Create user pool.Sau khi tạo thành công, lưu lại hai thông tin:
User pool ID
AWS Region
Các thông tin này sẽ được sử dụng khi cấu hình Frontend và JWT Authorizer trong Amazon API Gateway.
![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.2.png)

5. Tạo app client ->  **Create app client**
![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.3.png)

    Sau khi tọa thành công kiểm tra những thông tin và lưu những thông tin để deploy sau này
![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.4.png)
6. Sau cung edit Managed login pages configuration như sau:
    locahost để tiện cho việc phát triển 
    đuồng link cloudfront có thể upload sau để web hoạt động tốt
![endpoint](/images/5-Workshop/5.2/5.2.1/5.2.1.5.png)
