---
title: "Worklog Tuần 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Tìm hiểu cơ chế phân quyền Billing Console cho IAM User.
* Làm quen với IAM Identity Center và Identity Federation.
* Áp dụng Permission Boundaries để giới hạn quyền tối đa của IAM principal.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tìm hiểu Billing Console Delegation; tạo IAM User/Group và bật IAM access to billing. | 08/06/2026 | 08/06/2026 | <https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html> |
| 3 | Tạo Policy truy cập Billing, gán cho IAM principal và kiểm tra đăng nhập không sử dụng Root User. | 09/06/2026 | 09/06/2026 | <https://docs.aws.amazon.com/cost-management/latest/userguide/billing-permissions-ref.html> |
| 4 | Tìm hiểu IAM Identity Center, mô hình quản lý người dùng tập trung và Permission Set. | 10/06/2026 | 10/06/2026 | <https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html> |
| 5 | Tìm hiểu Identity Federation và quy trình cấp quyền truy cập nhiều AWS Account trong Organizations. | 11/06/2026 | 11/06/2026 | <https://docs.aws.amazon.com/singlesignon/latest/userguide/concepts.html> |
| 6 | Tạo IAM Permission Boundary, gán cho user/role và kiểm tra giới hạn quyền thực tế. | 12/06/2026 | 12/06/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html> |
| 7 và CN | So sánh Identity Policy, Permission Boundary và SCP; tổng hợp kết quả và dọn dẹp tài nguyên. | 13/06/2026 | 14/06/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html> |

### Kết quả đạt được tuần 8:

* Cho phép IAM User truy cập Billing Console mà không cần dùng Root User.
* Hiểu vai trò của IAM Identity Center trong quản lý truy cập tập trung.
* Phân biệt Identity Policy, Permission Boundary và Service Control Policy.
* Kiểm chứng Permission Boundary chỉ giới hạn quyền tối đa và không tự cấp quyền.

### Đánh giá cuối tuần:

* Các mục tiêu chính của tuần đã được hoàn thành theo kế hoạch.
* Nội dung thực hành giúp em củng cố kiến thức lý thuyết và rèn luyện khả năng triển khai trên AWS.
* Các khó khăn phát sinh được ghi nhận, kiểm tra và xử lý thông qua tài liệu AWS cùng quá trình trao đổi với mentor và thành viên nhóm.
* Em đã tổng hợp kết quả, rà soát tài nguyên và lưu lại các nội dung cần thiết để phục vụ báo cáo thực tập.
