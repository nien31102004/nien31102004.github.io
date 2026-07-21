---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# ASSUME ROLE VÀ SESSION TAGS TRONG QUẢN LÝ TRUY CẬP AWS

Khi hạ tầng AWS mở rộng sang nhiều tài khoản, việc quản lý quyền truy cập trở nên phức tạp hơn. AWS IAM cung cấp các cơ chế như Assume Role, IAM Conditions và Session Tags để triển khai mô hình phân quyền linh hoạt, an toàn và dựa trên thuộc tính (ABAC).

Những nội dung quan trọng cần nắm:

* IAM Role cho phép người dùng nhận thông tin xác thực tạm thời thông qua AWS Security Token Service (STS) bằng thao tác **AssumeRole**.
* Temporary Credentials giúp hạn chế sử dụng Access Key dài hạn, đồng thời hỗ trợ truy cập chéo tài khoản và phân quyền tạm thời một cách an toàn.
* IAM Conditions có thể giới hạn việc Assume Role theo địa chỉ IP (`aws:SourceIp`) hoặc thời gian truy cập (`aws:CurrentTime`), tăng cường bảo mật cho hệ thống.
* AWS IAM Identity Center hỗ trợ mô hình Attribute-Based Access Control (ABAC) bằng cách truyền các thuộc tính người dùng dưới dạng Session Tags trong quá trình xác thực.
* Session Tags cho phép cấp quyền động dựa trên việc so khớp thuộc tính của người dùng với thẻ của tài nguyên, giúp giảm đáng kể số lượng IAM Role cần quản lý.
* Mô hình này giúp đơn giản hóa việc quản trị quyền, tăng khả năng mở rộng và nâng cao mức độ bảo mật trong môi trường AWS nhiều tài khoản.

Việc kết hợp Assume Role, IAM Conditions, IAM Identity Center và Session Tags giúp triển khai hiệu quả nguyên tắc Least Privilege, đồng thời mang lại khả năng quản lý truy cập linh hoạt, bảo mật và dễ mở rộng.

### Hình ảnh
![](/images/3-blogsposted/blog2.png)

### Tham khảo

https://000044.awsstudygroup.com

https://aws.amazon.com/.../access-control-with-iam...

### Nguồn

https://www.facebook.com/share/p/14pJMQ8odjX/