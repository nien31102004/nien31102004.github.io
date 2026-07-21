---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Triển khai EBS Provisioned IOPS SSD `io2` và gắn vào EC2.
* Định dạng, mount volume và chia sẻ dữ liệu nội bộ bằng NFS.
* Kiểm tra khả năng đọc/ghi giữa các EC2 trong cùng VPC.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tạo hai EC2 làm NFS server và client; cấu hình Security Group cho SSH và NFS 2049. | 11/05/2026 | 11/05/2026 | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html> |
| 3 | Tạo EBS `io2` dung lượng 10 GiB, 1000 IOPS trong cùng Availability Zone với EC2 server. | 12/05/2026 | 12/05/2026 | <https://docs.aws.amazon.com/ebs/latest/userguide/provisioned-iops.html> |
| 4 | Attach volume vào EC2 server; kiểm tra thiết bị bằng `lsblk`, format `ext4` và mount tại `/mnt/shared`. | 13/05/2026 | 13/05/2026 | <https://docs.aws.amazon.com/ebs/latest/userguide/ebs-using-volumes.html> |
| 5 | Cài đặt NFS, cấu hình `/etc/exports`, khởi động dịch vụ và cấp quyền truy cập nội bộ. | 14/05/2026 | 14/05/2026 | <https://docs.aws.amazon.com/efs/latest/ug/mounting-fs-old.html> |
| 6 | Mount thư mục chia sẻ trên EC2 client; tạo, sửa và đọc tệp để kiểm tra đồng bộ dữ liệu. | 15/05/2026 | 15/05/2026 | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes-multi.html> |
| 7 và CN | Gỡ mount, xóa cấu hình NFS, detach và xóa EBS cùng các EC2 đã sử dụng. | 16/05/2026 | 17/05/2026 | <https://docs.aws.amazon.com/ebs/latest/userguide/ebs-deleting-volume.html> |

### Kết quả đạt được tuần 4:

* Triển khai thành công volume `io2` với IOPS cấu hình sẵn.
* Attach, format và mount volume ổn định trên EC2 server.
* Thiết lập NFS giúp EC2 client truy cập và ghi dữ liệu qua mạng nội bộ VPC.
* Hiểu rõ yêu cầu cùng Availability Zone của EBS và quy trình dọn dẹp tài nguyên.

### Đánh giá cuối tuần:

* Các mục tiêu chính của tuần đã được hoàn thành theo kế hoạch.
* Nội dung thực hành giúp em củng cố kiến thức lý thuyết và rèn luyện khả năng triển khai trên AWS.
* Các khó khăn phát sinh được ghi nhận, kiểm tra và xử lý thông qua tài liệu AWS cùng quá trình trao đổi với mentor và thành viên nhóm.
* Em đã tổng hợp kết quả, rà soát tài nguyên và lưu lại các nội dung cần thiết để phục vụ báo cáo thực tập.
