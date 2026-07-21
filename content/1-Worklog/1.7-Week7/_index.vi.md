---
title: "Worklog Tuần 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Tìm hiểu vai trò của AWS Tags trong quản lý và phân loại tài nguyên.
* Thực hành Resource Groups và thao tác tags trên nhiều EC2.
* Sử dụng AWS CLI để tự động hóa việc gắn và truy vấn tags.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tìm hiểu cấu trúc Key–Value của Tags và các trường hợp sử dụng theo Owner, Environment, Project, Service. | 01/06/2026 | 01/06/2026 | <https://docs.aws.amazon.com/tag-editor/latest/userguide/tagging.html> |
| 3 | Tạo hai EC2 tại Region Singapore và gán Tags khác nhau cho môi trường Test và UAT. | 02/06/2026 | 02/06/2026 | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html> |
| 4 | Chỉnh sửa, bổ sung và xóa Tags trên từng EC2; kiểm tra thay đổi trên AWS Console. | 03/06/2026 | 03/06/2026 | <https://docs.aws.amazon.com/tag-editor/latest/userguide/tagging.html> |
| 5 | Gắn Tags đồng thời cho nhiều tài nguyên; sử dụng bộ lọc Owner và Environment để tìm kiếm nhanh. | 04/06/2026 | 04/06/2026 | <https://docs.aws.amazon.com/ARG/latest/userguide/resource-groups.html> |
| 6 | Sử dụng AWS CLI với `create-tags`, `describe-instances` và bộ lọc `tag:` để quản lý tài nguyên. | 05/06/2026 | 05/06/2026 | <https://docs.aws.amazon.com/cli/latest/reference/ec2/create-tags.html> |
| 7 và CN | Tạo Resource Group dựa trên Tags; kiểm tra kết quả và dọn dẹp EC2 cùng cấu hình liên quan. | 06/06/2026 | 07/06/2026 | <https://docs.aws.amazon.com/ARG/latest/userguide/gettingstarted-query.html> |

### Kết quả đạt được tuần 7:

* Hiểu cách tổ chức tài nguyên AWS bằng Tags thống nhất.
* Gắn, chỉnh sửa, xóa và lọc tài nguyên theo Tags thành công.
* Tạo Resource Group dựa trên tiêu chí Tags.
* Sử dụng AWS CLI để hỗ trợ quản lý tài nguyên nhanh và nhất quán hơn.

### Đánh giá cuối tuần:

* Các mục tiêu chính của tuần đã được hoàn thành theo kế hoạch.
* Nội dung thực hành giúp em củng cố kiến thức lý thuyết và rèn luyện khả năng triển khai trên AWS.
* Các khó khăn phát sinh được ghi nhận, kiểm tra và xử lý thông qua tài liệu AWS cùng quá trình trao đổi với mentor và thành viên nhóm.
* Em đã tổng hợp kết quả, rà soát tài nguyên và lưu lại các nội dung cần thiết để phục vụ báo cáo thực tập.
