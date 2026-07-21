---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# XÂY DỰNG KIẾN TRÚC SERVERLESS VÀ GIÁM SÁT AN NINH TRÊN AWS

Kiến trúc Serverless giúp xây dựng ứng dụng đám mây có khả năng mở rộng tự động mà không cần quản lý hạ tầng máy chủ. Khi kết hợp với các giải pháp giám sát và phân tích an ninh, hệ thống có thể vận hành ổn định, tiết kiệm chi phí và tăng cường khả năng phòng chống các mối đe dọa.

Những nội dung quan trọng cần nắm:

* AWS Lambda cho phép thực thi mã nguồn theo sự kiện mà không cần quản lý máy chủ, đồng thời tự động mở rộng theo lưu lượng truy cập.
* Amazon S3 cung cấp dịch vụ lưu trữ đối tượng với độ bền cao và có thể kích hoạt Lambda thông qua Event Trigger để xây dựng các quy trình xử lý tự động.
* Amazon DynamoDB là cơ sở dữ liệu NoSQL serverless, tích hợp chặt chẽ với Lambda để lưu trữ dữ liệu ứng dụng theo mô hình on-demand.
* AWS IAM quản lý quyền truy cập giữa các dịch vụ AWS, đảm bảo giao tiếp an toàn theo nguyên tắc Least Privilege.
* GoAccess hỗ trợ trực quan hóa log máy chủ theo thời gian thực, giúp phát hiện nhanh các bất thường về lưu lượng và hoạt động đáng ngờ.
* Fail2Ban tự động phân tích log, phát hiện các hành vi tấn công như brute-force và chặn địa chỉ IP độc hại thông qua tường lửa.
* Wazuh là nền tảng SIEM giúp thu thập, phân tích và tương quan các sự kiện bảo mật từ nhiều nguồn, hỗ trợ giám sát tập trung và phát hiện sớm các mối đe dọa.

Việc kết hợp kiến trúc Serverless với các giải pháp giám sát, phân tích log và phòng thủ tự động giúp xây dựng hệ thống AWS có khả năng mở rộng, vận hành hiệu quả và đảm bảo an toàn trước các nguy cơ an ninh mạng.

### Hình ảnh
![](/images/3-blogsposted/blog3.png)

### Tham khảo

https://000078.awsstudygroup.com

https://aws.amazon.com/.../intelligent-failover-using.../

### Nguồn

https://www.facebook.com/share/p/18fuKNd7Hp/