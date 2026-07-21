---
title : "Quyết định"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

Trong phần này, chúng ta sẽ xây dựng **Decision Engine** để tổng hợp kết quả quét mã độc từ Amazon GuardDuty và kết quả phân tích nội dung từ AI, từ đó xác định trạng thái cuối cùng của từng tài liệu.

Decision Engine hỗ trợ ba hướng xử lý:

+ **ALLOW**: Tài liệu được xác định là an toàn và được chuyển từ prefix `quarantine/` sang `clean/`.
+ **REJECT**: Tài liệu chứa mã độc hoặc có mức độ rủi ro cao và được chuyển sang prefix `reject/`.
+ **REVIEW**: Tài liệu chưa đủ điều kiện để quyết định tự động, tiếp tục được giữ trong `quarantine/` để quản trị viên đánh giá thủ công.

Trong quá trình xử lý, AWS Lambda cập nhật trạng thái tài liệu trong Amazon DynamoDB, sao chép object đến prefix đích, xác minh object mới và chỉ xóa object nguồn sau khi quá trình sao chép hoàn tất.

Các nội dung thực hiện gồm:

1. [Tạo Lambda Decision Engine](5.7.1-create-decision-lambda/)
2. [Chuyển tài liệu an toàn](5.7.2-move-safe-file/)
3. [Chuyển tài liệu bị từ chối](5.7.3-move-reject-file/)

Sau khi hoàn thành, hệ thống có thể tự động phân loại tài liệu dựa trên kết quả bảo mật và phân tích AI, đồng thời hỗ trợ quản trị viên xử lý các tài liệu cần đánh giá thủ công.

{{% notice warning %}}
Nếu Amazon GuardDuty trả về `THREATS_FOUND`, tài liệu luôn được xử lý theo hướng `REJECT`. Kết quả AI hoặc quyết định thủ công không được phép ghi đè kết quả phát hiện mã độc.
{{% /notice %}}

{{% notice note %}}
Quá trình di chuyển tài liệu trong Amazon S3 được thực hiện bằng cách sao chép object sang prefix đích, xác minh object mới và sau đó xóa object cũ trong `quarantine/`.
{{% /notice %}}