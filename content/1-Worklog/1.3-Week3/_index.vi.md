---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Xây dựng VPC gồm public subnet và private subnet theo mô hình nhiều Availability Zone.
* Triển khai Amazon RDS for MySQL trong private subnet và giới hạn truy cập bằng Security Group.
* Kết nối EC2 với RDS, thực hiện snapshot, restore và dọn dẹp tài nguyên.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Thiết kế VPC CIDR `10.0.0.0/16`; tạo hai public subnet và hai private subnet ở hai Availability Zone. | 04/05/2026 | 04/05/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-getting-started.html> |
| 3 | Tạo Internet Gateway, public route table và bật auto-assign public IPv4 cho public subnet. | 05/05/2026 | 05/05/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html> |
| 4 | Tạo Security Group cho EC2 và RDS; chỉ cho phép MySQL 3306 từ Security Group của ứng dụng. | 06/05/2026 | 06/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html> |
| 5 | Tạo DB Subnet Group từ hai private subnet và triển khai RDS MySQL không public access. | 07/05/2026 | 07/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html> |
| 6 | Kết nối từ EC2 đến RDS; tạo database, bảng và dữ liệu mẫu để kiểm tra truy vấn. | 08/05/2026 | 08/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html> |
| 7 và CN | Tạo snapshot, restore RDS để kiểm tra khả năng khôi phục; sau đó dọn dẹp tài nguyên. | 09/05/2026 | 10/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html> |

### Kết quả đạt được tuần 3:

* Hoàn thiện VPC nhiều tầng với public/private subnet và định tuyến phù hợp.
* Triển khai RDS MySQL trong vùng mạng riêng, không cho phép truy cập trực tiếp từ Internet.
* Kết nối EC2 đến RDS thông qua Security Group và thực hiện truy vấn thành công.
* Thực hiện snapshot và restore, qua đó hiểu quy trình sao lưu và khôi phục cơ sở dữ liệu.

### Đánh giá cuối tuần:

* Các mục tiêu chính của tuần đã được hoàn thành theo kế hoạch.
* Nội dung thực hành giúp em củng cố kiến thức lý thuyết và rèn luyện khả năng triển khai trên AWS.
* Các khó khăn phát sinh được ghi nhận, kiểm tra và xử lý thông qua tài liệu AWS cùng quá trình trao đổi với mentor và thành viên nhóm.
* Em đã tổng hợp kết quả, rà soát tài nguyên và lưu lại các nội dung cần thiết để phục vụ báo cáo thực tập.
