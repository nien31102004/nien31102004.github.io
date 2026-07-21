---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# QUẢN TRỊ ĐỊNH DANH VÀ QUYỀN TRUY CẬP TRÊN AWS

Identity and Access Management (IAM) là nền tảng bảo mật quan trọng giúp kiểm soát ai được phép truy cập và thao tác trên tài nguyên AWS. Khi hệ thống phát triển theo mô hình nhiều tài khoản (multi-account), việc kết hợp nhiều loại IAM Policy sẽ giúp xây dựng cơ chế phòng thủ nhiều lớp (Defense-in-Depth) và tuân thủ nguyên tắc đặc quyền tối thiểu (Least Privilege).

Những nội dung quan trọng cần nắm:

* Hạn chế sử dụng tài khoản Root trong các tác vụ hằng ngày, thay vào đó nên ủy quyền quyền truy cập Billing cho IAM User khi cần thiết.
* AWS IAM Identity Center giúp quản lý tập trung người dùng và quyền truy cập giữa nhiều tài khoản AWS thông qua Single Sign-On (SSO), đồng thời hỗ trợ tự động hóa việc cấp quyền.
* Service Control Policies (SCPs) thiết lập giới hạn quyền tối đa trong AWS Organizations, giúp áp dụng các quy tắc bảo mật chung cho toàn bộ tổ chức.
* Resource Control Policies (RCPs) bảo vệ tài nguyên được hỗ trợ bằng cách giới hạn cách tài nguyên được chia sẻ hoặc truy cập, góp phần xây dựng Data Perimeter.
* Permissions Boundaries giới hạn quyền tối đa mà IAM User hoặc IAM Role có thể nhận được, giúp ngăn chặn việc leo thang đặc quyền khi phân quyền quản trị.
* Identity-based Policies được gắn vào IAM User, Group hoặc Role, trong khi Resource-based Policies được gắn trực tiếp lên tài nguyên như Amazon S3 Bucket hoặc AWS KMS Key.
* AWS sẽ đánh giá tất cả các policy liên quan trước khi cho phép một yêu cầu. Một lệnh **Explicit Deny** luôn có ưu tiên cao hơn mọi lệnh **Allow**.

Việc kết hợp IAM Identity Center cùng SCPs, RCPs, Permissions Boundaries, Identity-based Policies và Resource-based Policies giúp xây dựng mô hình bảo mật nhiều lớp, vừa đảm bảo khả năng quản trị tập trung vừa tăng cường mức độ an toàn cho hạ tầng AWS.

### Hình ảnh
![](/images/3-blogsposted/blog1.jpg)

### Tham khảo
https://aws.amazon.com/.../iam-policy-types-how-and-when...

https://000075.awsstudygroup.com

https://000030.awsstudygroup.com

https://000012.awsstudygroup.com

### Nguồn

https://www.facebook.com/share/p/18ArKBSHuD/