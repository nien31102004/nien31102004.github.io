---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---



# SECUREDOCS AI  
## Nền tảng lưu trữ và kiểm duyệt tài liệu an toàn tích hợp AI trên AWS

### 1. Tóm tắt điều hành

**SecureDocs AI** là nền tảng Web Application Serverless trên AWS, cho phép người dùng tải tài liệu lên, theo dõi trạng thái xử lý và chỉ lưu trữ tài liệu vào vùng an toàn sau khi hoàn thành quá trình kiểm tra bảo mật.

Hệ thống kết hợp **Amazon GuardDuty Malware Protection for S3** để phát hiện mã độc và **Amazon Bedrock Mantle** để phân tích nội dung tài liệu. Dựa trên kết quả từ hai lớp kiểm tra, **Decision Engine** tự động đưa ra một trong ba quyết định:

+ `ALLOW`: Tài liệu an toàn và được chuyển sang vùng `clean/`.
+ `REVIEW`: Tài liệu cần quản trị viên đánh giá thủ công và tiếp tục được giữ trong `quarantine/`.
+ `REJECT`: Tài liệu chứa mã độc hoặc có mức độ rủi ro cao và được chuyển sang `reject/`.

Người dùng được xác thực bằng **Amazon Cognito**, trong khi quyền truy cập API được bảo vệ bằng JWT Authorizer của Amazon API Gateway. Toàn bộ metadata, trạng thái quét, kết quả phân tích AI và quyết định cuối cùng được lưu trong Amazon DynamoDB.

Nền tảng hướng đến môi trường doanh nghiệp nhỏ, nhóm nghiên cứu hoặc hệ thống quản lý tài liệu nội bộ cần một quy trình kiểm duyệt tự động, có khả năng mở rộng và không phải vận hành máy chủ truyền thống.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Trong nhiều tổ chức, tài liệu nội bộ thường được chia sẻ qua email, ứng dụng trò chuyện hoặc các thư mục dùng chung. Quy trình này có thể dẫn đến các rủi ro:

+ Người dùng tải lên tài liệu chứa mã độc.
+ Tài liệu chứa nội dung phishing, social engineering hoặc yêu cầu đánh cắp thông tin xác thực.
+ Tài liệu chưa được kiểm tra vẫn được lưu chung với dữ liệu an toàn.
+ Trạng thái xử lý và lịch sử tài liệu không được theo dõi tập trung.
+ Quản trị viên khó xác định nguyên nhân tài liệu bị chặn hoặc cần đánh giá thủ công.
+ Việc mở rộng hệ thống yêu cầu quản lý thêm máy chủ, phần mềm quét và cơ chế phân quyền.

#### Giải pháp đề xuất

SecureDocs AI sử dụng kiến trúc AWS Serverless để tạo quy trình tải lên và kiểm duyệt tài liệu tự động:

1. Người dùng đăng nhập bằng Amazon Cognito.
2. Frontend gọi Amazon API Gateway để yêu cầu Presigned URL.
3. AWS Lambda tạo `documentId`, lưu metadata ban đầu vào DynamoDB và trả về Presigned URL.
4. Frontend tải tài liệu trực tiếp lên vùng `quarantine/` của Amazon S3.
5. Amazon GuardDuty Malware Protection for S3 quét object mới.
6. Kết quả GuardDuty được gửi qua Amazon EventBridge và Amazon SQS.
7. Nếu tài liệu có trạng thái `NO_THREATS_FOUND`, Lambda trích xuất nội dung và gửi đến Amazon Bedrock Mantle.
8. Decision Engine tổng hợp kết quả bảo mật và AI để đưa ra `ALLOW`, `REVIEW` hoặc `REJECT`.
9. Tài liệu được chuyển đến `clean/`, `reject/` hoặc tiếp tục nằm trong `quarantine/`.
10. Frontend truy vấn API để hiển thị trạng thái, kết quả và lịch sử xử lý.

#### Lợi ích và hoàn vốn đầu tư

Giải pháp mang lại các lợi ích:

+ Giảm thao tác kiểm tra tài liệu thủ công.
+ Ngăn tài liệu chưa được xác minh đi trực tiếp vào vùng lưu trữ an toàn.
+ Chuẩn hóa quy trình xử lý bằng các trạng thái có thể theo dõi.
+ Hỗ trợ quản trị viên đánh giá các trường hợp chưa rõ ràng.
+ Áp dụng kiến trúc Serverless giúp giảm công việc vận hành máy chủ.
+ Có thể mở rộng theo số lượng tài liệu và người dùng.
+ Tạo nền tảng để phát triển thêm chức năng tìm kiếm, báo cáo, kiểm toán và quản lý vòng đời tài liệu.

Hiệu quả đầu tư chủ yếu đến từ việc giảm thời gian kiểm tra thủ công, giảm nguy cơ phát tán tài liệu nguy hiểm và tăng khả năng truy vết khi xảy ra sự cố.

### 3. Kiến trúc giải pháp

SecureDocs AI sử dụng kiến trúc Serverless, Event-Driven và phân chia tài liệu thành ba vùng lưu trữ logic:

+ `quarantine/`: Tiếp nhận tài liệu mới và giữ tài liệu trong thời gian kiểm tra.
+ `clean/`: Lưu tài liệu đã được xác định là an toàn.
+ `reject/`: Lưu tài liệu bị phát hiện chứa mã độc hoặc có mức độ rủi ro cao.

![SecureDocs AI Architecture](/images/2-Proposal/securedocs-ai-architecture.png)

#### Luồng xử lý chính

1. **Authentication**: Người dùng đăng nhập qua Amazon Cognito và nhận JWT Token.
2. **Upload request**: Frontend gửi thông tin file đến API Gateway.
3. **Presigned URL**: Lambda kiểm tra tên file, định dạng, kích thước, tạo `documentId` và Presigned URL.
4. **Quarantine upload**: Frontend tải file trực tiếp lên Amazon S3 bằng phương thức `PUT`.
5. **Malware scanning**: GuardDuty Malware Protection for S3 quét object.
6. **Event processing**: EventBridge và SQS chuyển kết quả quét đến Lambda.
7. **AI analysis**: Lambda trích xuất nội dung TXT, PDF hoặc DOCX và gửi đến Bedrock Mantle.
8. **Decision**: Decision Engine tổng hợp kết quả và đưa ra `ALLOW`, `REVIEW` hoặc `REJECT`.
9. **Document movement**: Lambda sao chép object sang vùng đích, xác minh object mới rồi xóa object nguồn.
10. **Status retrieval**: Frontend gọi API để hiển thị trạng thái và lịch sử tài liệu.

#### Dịch vụ AWS sử dụng

+ **Amazon Cognito**: Xác thực người dùng và phân nhóm `Users`, `Admins`.
+ **Amazon API Gateway**: Cung cấp HTTP API và bảo vệ route bằng JWT Authorizer.
+ **AWS Lambda**: Tạo Presigned URL, xử lý kết quả quét, phân tích AI, Decision Engine và truy vấn tài liệu.
+ **Amazon S3**: Lưu Frontend và tài liệu trong các vùng `quarantine/`, `clean/`, `reject/`.
+ **Amazon CloudFront**: Phân phối Frontend qua HTTPS.
+ **AWS WAF**: Bảo vệ CloudFront và giới hạn các request không hợp lệ.
+ **Amazon GuardDuty Malware Protection for S3**: Quét mã độc cho object mới.
+ **Amazon EventBridge**: Nhận sự kiện kết quả quét từ GuardDuty.
+ **Amazon SQS**: Tách rời các thành phần và hỗ trợ xử lý bất đồng bộ.
+ **Amazon Bedrock Mantle**: Phân tích nội dung và đánh giá rủi ro tài liệu.
+ **Amazon DynamoDB**: Lưu metadata, trạng thái, kết quả quét và quyết định cuối cùng.
+ **AWS Secrets Manager**: Lưu API Key của Bedrock Mantle.
+ **AWS KMS**: Mã hóa dữ liệu lưu trữ.
+ **Amazon CloudWatch**: Thu thập log, metric và cảnh báo.
+ **AWS CloudTrail**: Ghi nhận hoạt động quản trị và API.
+ **AWS IAM**: Áp dụng nguyên tắc least privilege cho từng thành phần.

#### Thiết kế thành phần

+ **Frontend**: HTML, CSS và JavaScript, triển khai trên Amazon S3 và CloudFront.
+ **Authentication**: Cognito User Pool, App Client, Hosted UI và PKCE.
+ **Upload Service**: API Gateway gọi Lambda để tạo Presigned URL có thời hạn.
+ **Storage Layer**: Amazon S3 quản lý nội dung file; DynamoDB quản lý metadata.
+ **Malware Layer**: GuardDuty quét file trước khi phân tích AI.
+ **AI Layer**: Lambda trích xuất văn bản và gửi nội dung đến Bedrock Mantle.
+ **Decision Layer**: Tổng hợp kết quả, cập nhật DynamoDB và di chuyển tài liệu.
+ **Admin Review**: Quản trị viên xử lý tài liệu ở trạng thái `MANUAL_REVIEW`.

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai

Dự án được triển khai theo các giai đoạn:

1. **Phân tích yêu cầu và thiết kế kiến trúc**
   + Xác định luồng người dùng, loại file, giới hạn dung lượng và trạng thái tài liệu.
   + Thiết kế kiến trúc Serverless và luồng xử lý Event-Driven.

2. **Xây dựng lớp xác thực và lưu trữ**
   + Tạo Cognito User Pool, App Client và nhóm người dùng.
   + Tạo S3 storage, DynamoDB table, prefix và CORS.

3. **Xây dựng chức năng Upload**
   + Tạo Lambda cấp Presigned URL.
   + Tạo route `POST /upload-url`.
   + Kiểm tra upload từ Frontend vào `quarantine/`.

4. **Xây dựng Malware Scanning**
   + Bật GuardDuty Malware Protection for S3.
   + Cấu hình EventBridge và SQS.
   + Xử lý và lưu kết quả quét vào DynamoDB.

5. **Xây dựng AI Content Analysis**
   + Chuẩn bị thư viện `pypdf` và `python-docx`.
   + Lưu Mantle API Key trong Secrets Manager.
   + Trích xuất nội dung và phân tích rủi ro.

6. **Xây dựng Decision Engine**
   + Áp dụng luật ưu tiên GuardDuty.
   + Xử lý `ALLOW`, `REVIEW` và `REJECT`.
   + Xác minh object đích trước khi xóa object nguồn.

7. **Hoàn thiện Frontend và kiểm thử**
   + Hiển thị danh sách, trạng thái và chi tiết tài liệu.
   + Kiểm thử tài liệu an toàn, tài liệu bị từ chối và tài liệu cần review.
   + Triển khai Frontend lên CloudFront.

#### Yêu cầu kỹ thuật

+ **AWS Region chính**: `ap-southeast-1`.
+ **Bedrock Mantle Region**: `ap-northeast-1`.
+ **Lambda Runtime**: Python 3.14.
+ **Frontend**: HTML, CSS và JavaScript.
+ **Định dạng hỗ trợ giai đoạn đầu**: `.txt`, `.pdf`, `.docx`.
+ **Kích thước file tối đa**: 10 MB.
+ **Thời hạn Presigned URL**: 300 giây.
+ **Bảo mật S3**: Block Public Access, mã hóa SSE-KMS và truy cập qua IAM hoặc Presigned URL.
+ **API Security**: JWT Authorizer và kiểm tra quyền nhóm Cognito.
+ **Xử lý sự kiện**: Thiết kế idempotent để hạn chế xử lý trùng.
+ **Giám sát**: CloudWatch Logs, metric, alarm và CloudTrail.

### 5. Lộ trình và mốc triển khai

+ **Giai đoạn 1 – Phân tích và thiết kế**
  + Hoàn thiện yêu cầu, sơ đồ kiến trúc và mô hình dữ liệu.

+ **Giai đoạn 2 – Authentication, Storage và Upload**
  + Hoàn thiện Cognito, S3, DynamoDB, API Gateway và Presigned URL.

+ **Giai đoạn 3 – Malware Scanning và AI Analysis**
  + Tích hợp GuardDuty, EventBridge, SQS, Bedrock Mantle và Secrets Manager.

+ **Giai đoạn 4 – Decision Engine và Frontend**
  + Hoàn thiện luồng `ALLOW`, `REVIEW`, `REJECT`, giao diện lịch sử và chi tiết tài liệu.

+ **Giai đoạn 5 – Kiểm thử và triển khai**
  + Kiểm thử End-to-End, phân quyền, CORS, lỗi dịch vụ, bảo mật và hiệu năng.
  + Triển khai Frontend lên CloudFront và hoàn thiện tài liệu workshop.

+ **Sau triển khai**
  + Bổ sung chức năng tìm kiếm, tải xuống an toàn, xóa tài liệu, thông báo và báo cáo quản trị.

### 6. Ước tính ngân sách

Chi phí phụ thuộc vào số lượng tài liệu, kích thước file, lưu lượng CloudFront, dung lượng GuardDuty quét và số token AI sử dụng.

Các nhóm chi phí chính:

+ Amazon S3 và CloudFront.
+ AWS Lambda và Amazon API Gateway.
+ Amazon DynamoDB.
+ Amazon SQS và Amazon EventBridge.
+ GuardDuty Malware Protection for S3.
+ Amazon Bedrock Mantle.
+ AWS Secrets Manager và AWS KMS.
+ Amazon CloudWatch, CloudTrail và AWS WAF.

Đối với môi trường demo có số lượng người dùng và tài liệu thấp, ngân sách dự kiến nên được đặt trong khoảng:

```text
5–30 USD/tháng
```
![Secure AI-Driven Document Platform Architecture Overview](/images/2-Proposal/kientrucaws.png)

#### Video demo
https://drive.google.com/drive/folders/107I3wMFQx0EnR0jGuWC703YX0urXKBPx

#### Link website
https://d2aoitb7o6jdem.cloudfront.net/login