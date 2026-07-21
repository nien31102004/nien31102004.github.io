---

title : "Giới thiệu"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
-----------------------

#### Giới thiệu về dự án

Secure AI-Driven Document Platform là nền tảng lưu trữ và kiểm duyệt tài liệu an toàn được xây dựng trên nền tảng AWS. Hệ thống cho phép người dùng tải tài liệu lên, theo dõi trạng thái xử lý và chỉ truy cập những tài liệu đã vượt qua quá trình kiểm tra bảo mật.

Dự án được xây dựng theo kiến trúc Serverless kết hợp với mô hình Event-Driven. Các thành phần như Amazon API Gateway, AWS Lambda, Amazon S3, Amazon SQS và Amazon DynamoDB phối hợp với nhau để xử lý tài liệu tự động mà không cần duy trì máy chủ truyền thống.

Sau khi người dùng tải tài liệu lên, hệ thống sẽ thực hiện hai quá trình kiểm tra chính:

* Amazon GuardDuty Malware Protection kiểm tra tài liệu nhằm phát hiện mã độc và các mối đe dọa bảo mật.
* Mô hình AI thông qua Bedrock Mantle phân tích nội dung tài liệu nhằm phát hiện dấu hiệu lừa đảo, Social Engineering hoặc các nội dung có mức độ rủi ro cao.

Dựa trên kết quả kiểm tra, hệ thống sẽ đánh giá mức độ an toàn, cập nhật trạng thái xử lý và chuyển tài liệu đến vùng lưu trữ phù hợp.

#### Mục tiêu của dự án

Dự án được triển khai nhằm giải quyết các vấn đề bảo mật trong quá trình tải lên và lưu trữ tài liệu trực tuyến, bao gồm:

* Ngăn chặn tài liệu chứa mã độc trước khi người dùng có thể tải xuống hoặc sử dụng.
* Phát hiện các nội dung có dấu hiệu Phishing hoặc Social Engineering bằng AI.
* Xác thực và phân quyền người dùng bằng Amazon Cognito.
* Mã hóa tài liệu trong quá trình lưu trữ và truyền tải.
* Tự động hóa quá trình kiểm duyệt tài liệu bằng các sự kiện và hàng đợi.
* Lưu lại trạng thái, điểm rủi ro và kết quả phân tích để phục vụ tra cứu và quản trị.
* Xây dựng hệ thống có khả năng mở rộng linh hoạt và tối ưu chi phí vận hành.

#### Tổng quan về hệ thống

Trong dự án này, hệ thống được chia thành các thành phần chính sau:

* **Frontend Application** cung cấp giao diện đăng nhập, tải tài liệu, quản lý tài liệu và xem kết quả kiểm tra. Frontend được xây dựng bằng Vite và triển khai dưới dạng Static Website thông qua Amazon S3 và Amazon CloudFront.

* **Amazon Cognito** quản lý tài khoản, xác thực người dùng và cung cấp JWT Token. Token được gửi kèm trong các yêu cầu API để xác định người dùng và kiểm tra quyền truy cập.

* **Amazon API Gateway** cung cấp các API cho Frontend, bao gồm tạo Presigned URL, lấy danh sách tài liệu, xem chi tiết tài liệu, xóa tài liệu, lấy kết quả quét và tạo đường dẫn tải xuống.

* **AWS Lambda** xử lý logic nghiệp vụ của hệ thống như tạo Presigned URL, truy vấn tài liệu, xử lý kết quả quét, phân tích AI và đưa ra quyết định cuối cùng đối với tài liệu.

* **Amazon S3** lưu trữ tài liệu trong một bucket duy nhất và tổ chức theo các prefix khác nhau:

  * `quarantine/` lưu tài liệu vừa được tải lên và đang chờ kiểm tra.
  * `clean/` lưu tài liệu đã được xác định là an toàn.
  * `reject/` lưu tài liệu bị từ chối do phát hiện mã độc hoặc nội dung có rủi ro.

* **Amazon SQS** tiếp nhận các yêu cầu xử lý tài liệu và kết quả kiểm tra. Hàng đợi giúp tách biệt quá trình tải tài liệu khỏi quá trình quét, hạn chế mất dữ liệu và hỗ trợ xử lý bất đồng bộ.

* **Amazon GuardDuty Malware Protection** thực hiện kiểm tra mã độc đối với tài liệu được tải lên vùng `quarantine/`.

* **Bedrock Mantle** gọi mô hình AI để phân tích nội dung được trích xuất từ các tài liệu PDF, TXT hoặc DOCX. Kết quả phân tích bao gồm mức độ rủi ro, điểm rủi ro, nội dung tóm tắt và hành động được đề xuất.

* **Amazon DynamoDB** lưu trữ metadata của tài liệu, bao gồm tên tài liệu, người sở hữu, vị trí lưu trữ, trạng thái xử lý, kết quả quét mã độc, kết quả AI, điểm rủi ro và thời gian cập nhật.

* **Amazon CloudWatch** lưu log và hỗ trợ theo dõi hoạt động của các hàm Lambda trong toàn bộ quá trình xử lý tài liệu.

#### Tổng quan về luồng xử lý tài liệu

Quá trình xử lý tài liệu trong hệ thống được thực hiện theo các bước chính sau:

1. Người dùng đăng nhập vào hệ thống thông qua Amazon Cognito và nhận JWT Token.

2. Frontend gửi yêu cầu đến API Gateway để lấy Presigned URL dành cho việc tải tài liệu.

3. Lambda tạo mã `documentId`, lưu thông tin ban đầu vào DynamoDB và trả Presigned URL cho Frontend.

4. Frontend sử dụng Presigned URL để tải tài liệu trực tiếp lên Amazon S3 tại prefix `quarantine/`.

5. Sau khi tài liệu được tải lên thành công, Amazon S3 phát sinh sự kiện `ObjectCreated` và gửi thông tin tài liệu vào Amazon SQS.

6. Lambda xử lý message từ SQS, cập nhật trạng thái tài liệu và bắt đầu quá trình kiểm tra.

7. Amazon GuardDuty Malware Protection quét tài liệu để phát hiện mã độc.

8. Kết quả quét GuardDuty được gửi thông qua Amazon EventBridge và Amazon SQS đến Lambda xử lý kết quả.

9. Nếu không phát hiện mã độc, nội dung tài liệu được trích xuất và gửi đến mô hình AI thông qua Bedrock Mantle để phân tích.

10. Lambda Decision Engine tổng hợp kết quả từ GuardDuty và AI để xác định hành động cuối cùng.

11. Tài liệu an toàn được chuyển từ prefix `quarantine/` sang `clean/`. Tài liệu nguy hiểm hoặc có mức độ rủi ro cao được chuyển sang `reject/`.

12. Trạng thái cuối cùng, kết quả quét, điểm rủi ro và lý do xử lý được lưu vào Amazon DynamoDB.

13. Frontend gọi API để lấy kết quả và hiển thị trạng thái tài liệu cho người dùng.

#### Các trạng thái xử lý tài liệu

Trong quá trình hoạt động, một tài liệu có thể trải qua các trạng thái sau:

* `UPLOADED`: Tài liệu đã được tải lên Amazon S3 thành công.
* `SCANNING`: Tài liệu đang được kiểm tra mã độc hoặc phân tích nội dung.
* `AI_ANALYSIS_COMPLETED`: Quá trình phân tích nội dung bằng AI đã hoàn thành.
* `SAFE`: Tài liệu an toàn và đã được chuyển sang vùng lưu trữ `clean/`.
* `REJECTED`: Tài liệu bị từ chối do phát hiện mã độc hoặc nội dung có mức độ rủi ro cao.
* `ERROR`: Quá trình xử lý tài liệu gặp lỗi và cần được kiểm tra lại.

#### Kết quả mong đợi

Sau khi hoàn thành dự án, hệ thống có thể:

* Xác thực người dùng và bảo vệ các API bằng JWT Token.
* Tạo Presigned URL để người dùng tải tài liệu trực tiếp lên Amazon S3.
* Tự động kích hoạt quá trình kiểm tra sau khi tài liệu được tải lên.
* Phát hiện mã độc bằng Amazon GuardDuty Malware Protection.
* Trích xuất và phân tích nội dung tài liệu bằng AI.
* Đưa ra điểm rủi ro và hành động xử lý phù hợp.
* Phân loại tài liệu vào vùng an toàn hoặc vùng từ chối.
* Lưu lại toàn bộ metadata và kết quả xử lý trong Amazon DynamoDB.
* Cho phép người dùng xem danh sách, trạng thái, kết quả kiểm tra và tải xuống tài liệu an toàn.
* Cho phép quản trị viên theo dõi tài liệu, xem kết quả kiểm duyệt và thực hiện đánh giá khi cần thiết.

![Tổng quan kiến trúc Secure AI-Driven Document Platform](/images/2-Proposal/kientrucaws.png)

#### Video demo
https://drive.google.com/drive/folders/107I3wMFQx0EnR0jGuWC703YX0urXKBPx

#### Link website
https://d2aoitb7o6jdem.cloudfront.net/login