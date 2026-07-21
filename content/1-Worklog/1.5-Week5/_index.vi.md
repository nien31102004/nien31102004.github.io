---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Nắm vững IAM User, Group, Role, Policy và luồng xử lý yêu cầu cấp quyền.
* Thực hành Assume Role, Switch Role và Trust Relationship.
* Áp dụng IAM Condition để giới hạn quyền theo địa chỉ IP và thời gian.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Ôn tập cấu trúc IAM request gồm Principal, Action, Resource, Environment và Policy evaluation. | 18/05/2026 | 18/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html> |
| 3 | Tạo IAM Group và bốn IAM User với các mức quyền EC2, RDS, kế thừa từ Group và không có quyền. | 19/05/2026 | 19/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html> |
| 4 | Đăng nhập bằng từng IAM User để kiểm tra quyền được cấp và xác nhận các trường hợp AccessDenied. | 20/05/2026 | 20/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html> |
| 5 | Tạo IAM Role có quyền quản trị; chỉnh Trust Relationship và cho phép user không có quyền thực hiện Assume Role. | 21/05/2026 | 21/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-console.html> |
| 6 | Thêm Condition giới hạn Assume Role theo dải IP và kiểm tra trường hợp hợp lệ/không hợp lệ. | 22/05/2026 | 22/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html> |
| 7 và CN | Thêm Condition theo thời gian, tổng hợp kết quả kiểm thử và xóa User, Group, Role, Policy đã tạo. | 23/05/2026 | 24/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws-dates.html> |

### Kết quả đạt được tuần 5:

* Phân biệt rõ IAM User, Group, Role, Identity Policy và Trust Policy.
* Thực hiện thành công Switch Role và sử dụng temporary credentials thông qua AWS STS.
* Kiểm chứng Condition theo IP và thời gian có thể giới hạn Assume Role.
* Củng cố tư duy phân quyền theo nguyên tắc đặc quyền tối thiểu.

### Đánh giá cuối tuần:

* Các mục tiêu chính của tuần đã được hoàn thành theo kế hoạch.
* Nội dung thực hành giúp em củng cố kiến thức lý thuyết và rèn luyện khả năng triển khai trên AWS.
* Các khó khăn phát sinh được ghi nhận, kiểm tra và xử lý thông qua tài liệu AWS cùng quá trình trao đổi với mentor và thành viên nhóm.
* Em đã tổng hợp kết quả, rà soát tài nguyên và lưu lại các nội dung cần thiết để phục vụ báo cáo thực tập.
