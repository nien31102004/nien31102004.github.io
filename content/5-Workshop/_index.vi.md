---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Nền tảng tài liệu an toàn ứng dụng AI

Trong workshop này, bạn sẽ xây dựng một nền tảng serverless trên AWS cho phép tải tài liệu lên, quét mã độc, phân tích nội dung bằng AI và chỉ phát hành những tài liệu đáp ứng chính sách bảo mật.

Giải pháp kết hợp API đồng bộ với quy trình xử lý hướng sự kiện. Người dùng và quản trị viên thao tác qua ứng dụng React; phần backend sử dụng các dịch vụ được quản lý gồm Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon S3, Amazon DynamoDB, Amazon SQS, Amazon EventBridge, Amazon GuardDuty Malware Protection, Amazon Bedrock Mantle, Amazon CloudFront và Amazon CloudWatch.


## Hệ thống bạn sẽ xây dựng

Sau khi hoàn thành workshop, nền tảng sẽ có:

- Đăng nhập bằng Cognito cho người dùng thông thường và quản trị viên.
- API người dùng và API quản trị được bảo vệ bằng JWT.
- Tải trực tiếp từ trình duyệt lên S3 qua Presigned URL có thời hạn ngắn.
- Các vùng lưu trữ riêng biệt cho tài liệu cách ly, an toàn và bị từ chối.
- Tự động quét mã độc bằng GuardDuty Malware Protection for S3.
- Phân tích AI để phát hiện nguy cơ phishing, gian lận, social engineering, đánh cắp thông tin xác thực và prompt injection.
- Decision Engine xác định kết quả cuối cùng từ kết quả mã độc, AI và đánh giá của quản trị viên.
- API cho phép người dùng liệt kê, xem, tải xuống và xóa tài liệu.
- Quy trình quản trị để đánh giá tài liệu, quản lý tài liệu và quản lý người dùng.
- Frontend React được lưu trữ riêng tư trong S3 và phân phối qua CloudFront bằng HTTPS.
- Bộ kiểm thử đầu cuối, log vận hành và quy trình dọn dẹp đầy đủ.

## Tổng quan kiến trúc

![Kiến trúc Secure AI-Driven Document Platform](/images/2-Proposal/kientrucaws.png)

| Lớp | Dịch vụ AWS | Trách nhiệm |
|---|---|---|
| Truy cập người dùng | CloudFront, Amazon S3, Amazon Cognito | Phân phối ứng dụng web và xác thực người dùng |
| API | Amazon API Gateway, AWS Lambda | Phân quyền yêu cầu và xử lý chức năng người dùng, quản trị |
| Lưu trữ tài liệu | Amazon S3 | Cô lập tài liệu mới tải lên, đã duyệt và bị từ chối |
| Điều phối | Amazon SQS, Amazon EventBridge, DynamoDB Streams | Kết nối các giai đoạn xử lý mà không phụ thuộc chặt chẽ |
| Phân tích bảo mật | GuardDuty Malware Protection, Bedrock Mantle | Phát hiện mã độc và phân tích rủi ro nội dung |
| Trạng thái | Amazon DynamoDB | Lưu chủ sở hữu, trạng thái, kết quả quét, kết quả AI và quyết định |
| Vận hành | Amazon CloudWatch | Lưu log, hỗ trợ khắc phục sự cố và xác minh hệ thống |

## Luồng xử lý tài liệu

1. Người dùng đăng nhập qua Amazon Cognito và nhận token cho phiên làm việc trên web.
2. Frontend gọi API tải lên đã được bảo vệ bằng access token của Cognito.
3. Lambda tải lên tạo `documentId`, ghi bản ghi ban đầu vào DynamoDB và trả về S3 Presigned URL có thời hạn ngắn.
4. Trình duyệt tải trực tiếp tài liệu vào prefix `uploads/` của bucket cách ly.
5. Amazon S3 gửi sự kiện tải lên đến Amazon SQS, đồng thời GuardDuty Malware Protection quét object mới.
6. Kết quả GuardDuty được chuyển qua Amazon EventBridge đến Lambda xử lý kết quả quét. Lambda chuẩn hóa kết quả và cập nhật DynamoDB.
7. Chỉ tài liệu không có mã độc mới được trích xuất văn bản và phân tích nội dung bằng AI qua Bedrock Mantle.
8. DynamoDB Streams kích hoạt Decision Engine khi có kết quả mã độc, AI hoặc đánh giá của quản trị viên.
9. Tài liệu an toàn được sao chép sang bucket clean; tài liệu bị chặn được sao chép sang bucket rejected. Bản nguồn chỉ bị xóa sau khi đã xác minh bản đích.
10. Frontend đọc trạng thái hiện tại qua Document API. Tài liệu cần con người đánh giá sẽ giữ trạng thái `MANUAL_REVIEW` cho đến khi quản trị viên được cấp quyền gửi quyết định.

Tài liệu đã xác nhận có mã độc luôn bị từ chối và không thể bị thay đổi kết quả bởi AI hoặc quản trị viên.

## Lộ trình workshop

Hãy hoàn thành các phần theo đúng thứ tự vì mỗi phần sử dụng tài nguyên được tạo ở phần trước.

| Phần | Nội dung thực hiện |
|---|---|
| [5.1 Tổng quan workshop](5.1-workshop-overview/) | Tìm hiểu bài toán, kiến trúc, mục tiêu và kết quả mong đợi |
| [5.2 Xác thực](5.2-authentication/) | Tạo Cognito user pool, app client, group và tài khoản kiểm thử |
| [5.3 Lưu trữ](5.3-storage/) | Tạo các S3 bucket, bảng DynamoDB, prefix và cấu hình CORS |
| [5.4 Tải tài liệu](5.4-upload/) | Xây dựng Lambda và API tạo Presigned URL, sau đó thử tải tài liệu |
| [5.5 Quét mã độc](5.5-malware-scan/) | Cấu hình SQS, sự kiện S3, GuardDuty và xử lý kết quả quét |
| [5.6 Phân tích AI](5.6-ai-analysis/) | Chuẩn bị thư viện trích xuất, cấu hình Mantle và phân tích tài liệu an toàn |
| [5.7 Ra quyết định](5.7-decision/) | Tạo Decision Engine và chuyển tài liệu đến vùng clean hoặc rejected |
| [5.8 Document API](5.8-document-api/) | Thêm API liệt kê, xem, xóa, tải xuống và lấy kết quả quét |
| [5.9 Quản trị](5.9-admin/) | Thêm chức năng đánh giá tài liệu và quản lý người dùng có phân quyền |
| [5.10 Frontend](5.10-frontend/) | Xây dựng ứng dụng Vite và triển khai qua S3, CloudFront |
| [5.11 Kiểm thử](5.11-testing/) | Kiểm tra bảo mật, các nhánh xử lý, retry và hành vi trên trình duyệt |
| [5.12 Dọn dẹp](5.12-cleanup/) | Xóa tài nguyên workshop an toàn và xác minh không còn tài nguyên tính phí |

## Nguyên tắc bảo mật xuyên suốt workshop

- Trình duyệt không bao giờ nhận AWS access key hoặc secret của ứng dụng.
- API Gateway xác thực JWT; Lambda kiểm tra quyền sở hữu tài liệu và quyền Admin ở phía máy chủ.
- Presigned URL có thời hạn ngắn và chỉ cho phép đúng thao tác S3 trên đúng object key.
- Tài liệu mới tải lên phải ở vùng cách ly cho đến khi hoàn thành mọi bước kiểm tra bắt buộc.
- Nội dung trích xuất từ tài liệu được xem là dữ liệu không đáng tin cậy; AI chỉ phân loại và không làm theo chỉ dẫn nằm trong tài liệu.
- Conditional update của DynamoDB và handler có tính idempotent bảo vệ hệ thống trước sự kiện trùng lặp và quyết định cạnh tranh.
- Decision Engine xác minh bản sao ở đích trước khi xóa bản nguồn trong vùng cách ly.
- Log chỉ chứa mã định danh và chuyển đổi trạng thái; không ghi token, secret, toàn bộ nội dung tài liệu hoặc Presigned URL.

## Chuẩn bị trước khi bắt đầu

Bạn cần chuẩn bị:

- Tài khoản AWS hoặc workshop sandbox có quyền tạo các dịch vụ được sử dụng trong tài liệu.
- Quyền truy cập AWS Management Console và terminal đã cấu hình AWS CLI cho những bước dùng dòng lệnh.
- Môi trường phát triển cục bộ để tạo Lambda package và frontend Vite.
- Trình duyệt hiện đại để kiểm thử Cognito, API, chức năng tải lên và CloudFront.
- Các tệp TXT, PDF và DOCX mẫu không chứa thông tin nhạy cảm. Chỉ sử dụng tệp kiểm thử chống mã độc vô hại được hướng dẫn trong phần quét mã độc; tuyệt đối không dùng mã độc thật.

Trừ khi từng phần có chỉ dẫn khác, hãy triển khai workload chính tại `ap-southeast-1`. Luôn kiểm tra Region hiển thị trong từng bước, đặc biệt khi cấu hình thông tin xác thực Bedrock Mantle và các dịch vụ toàn cục như CloudFront, IAM.

Ghi lại các giá trị được tạo như tên bucket, URL của API, Cognito ID, tên Lambda và CloudFront domain. Các phần sau sẽ tiếp tục sử dụng những giá trị này.

## Tiêu chí hoàn thành

Workshop hoàn tất khi bạn chứng minh được:

- Người dùng thông thường có thể đăng nhập, tải tài liệu an toàn, theo dõi trạng thái và chỉ tải xuống sau khi quyết định cuối cùng là `SAFE`.
- Tài liệu có mã độc hoặc rủi ro cao bị từ chối và không thể tải xuống.
- Người dùng không thể xem, xóa hoặc tải xuống tài liệu của người khác.
- Quản trị viên có thể đánh giá tài liệu phù hợp nhưng không thể ghi đè kết quả đã xác nhận có mã độc.
- Sự kiện trùng lặp và quyết định đồng thời không tạo metadata mâu thuẫn hoặc nhiều bản tài liệu có hiệu lực.
- Ứng dụng hoạt động qua CloudFront HTTPS trong khi bucket S3 chứa frontend vẫn riêng tư.
- CloudWatch Logs cho phép truy vết từng kiểm thử mà không để lộ thông tin xác thực hoặc nội dung nhạy cảm.
- Toàn bộ tài nguyên workshop được xóa sau khi xác minh lần cuối.

Chọn **5.1 Tổng quan workshop** để bắt đầu.
