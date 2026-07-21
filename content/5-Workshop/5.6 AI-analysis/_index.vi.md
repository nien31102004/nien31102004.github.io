---
title : "Phân tích nội dung bằng AI"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

Trong phần này, chúng ta sẽ xây dựng quy trình phân tích nội dung tài liệu bằng AI cho hệ thống **Secure AI-Driven Document Platform**.

Sau khi Amazon GuardDuty xác nhận tài liệu không chứa mã độc với trạng thái `NO_THREATS_FOUND`, AWS Lambda tải tài liệu từ Amazon S3, trích xuất nội dung văn bản và gửi dữ liệu đến Amazon Bedrock Mantle để đánh giá mức độ rủi ro.

Hệ thống hỗ trợ trích xuất nội dung từ các định dạng:

+ **TXT**: Giải mã trực tiếp bằng Python.
+ **PDF**: Trích xuất bằng thư viện `pypdf`.
+ **DOCX**: Trích xuất bằng thư viện `python-docx`.

Kết quả phân tích AI được chuẩn hóa dưới dạng JSON và lưu vào Amazon DynamoDB, bao gồm điểm rủi ro, mức độ rủi ro, nội dung tóm tắt, độ tin cậy và hành động được đề xuất như `ALLOW`, `REVIEW` hoặc `REJECT`.

Các nội dung thực hiện gồm:

1. [Chuẩn bị thư viện trích xuất tài liệu](5.6.1-prepare-document-libraries/)
2. [Cấu hình Amazon Bedrock Mantle](5.6.2-configure-bedrock-mantle/)
3. [Tạo Lambda phân tích nội dung bằng AI](5.6.3-create-ai-analysis-lambda/)
4. [Kiểm tra phân tích tài liệu bằng AI](5.6.4-test-ai-analysis/)

Sau khi hoàn thành, hệ thống có thể tự động trích xuất nội dung tài liệu, phân tích các dấu hiệu như phishing, social engineering, đánh cắp thông tin xác thực, giả mạo, gian lận thanh toán và prompt injection, sau đó chuyển kết quả đến Decision Engine để xử lý bước tiếp theo.

{{% notice warning %}}
Chỉ gửi nội dung tài liệu sang AI sau khi Amazon GuardDuty trả về `NO_THREATS_FOUND`. Nếu phát hiện `THREATS_FOUND`, tài liệu không được gửi đến Bedrock Mantle.
{{% /notice %}}

{{% notice note %}}
Bedrock Mantle API Key phải được lưu trong AWS Secrets Manager. Không lưu trực tiếp API Key trong mã nguồn Lambda hoặc Environment variables.
{{% /notice %}}