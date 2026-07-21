---
title: "Chia sẻ, đóng góp ý kiến"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

### Đánh giá chung

**1. Môi trường học tập và làm việc**  
Trong thời gian tham gia chương trình First Cloud AI Journey, em học tập và thực tập theo hình thức hybrid. Hình thức này giúp em chủ động thực hiện các bài lab AWS từ xa, đồng thời vẫn có thể tham gia các buổi hướng dẫn trực tiếp, seminar và hoạt động nhóm. Môi trường học tập khá cởi mở, khuyến khích sinh viên đặt câu hỏi, chia sẻ khó khăn và hỗ trợ lẫn nhau. Nhờ quá trình này, em dần tự tin hơn khi thao tác với các dịch vụ AWS và phối hợp cùng các thành viên khác.

**2. Sự hỗ trợ của mentor và team admin**  
Mentor và team admin đã cung cấp tài liệu, bài tập theo tuần, hướng dẫn kỹ thuật và các thông báo cần thiết trong suốt kỳ thực tập. Khi gặp lỗi trong quá trình cấu hình dịch vụ AWS, em được khuyến khích xem lại luồng hệ thống, đọc thông báo lỗi và tự tìm nguyên nhân trước khi nhờ hỗ trợ. Cách hướng dẫn này giúp em nâng cao khả năng tự học và xử lý vấn đề, thay vì chỉ phụ thuộc vào các bước có sẵn.

**3. Sự phù hợp với chuyên ngành học**  
Nội dung thực tập có sự liên quan rõ rệt đến ngành Công nghệ thông tin và chuyên ngành Công nghệ phần mềm mà em đang theo học. Em có cơ hội áp dụng các kiến thức về phát triển backend, API, xác thực, phân quyền, cơ sở dữ liệu, bảo mật và thiết kế hệ thống. Bên cạnh đó, em còn được tiếp cận nhiều nội dung mới chưa được học sâu tại trường như IAM Role và Policy, Amazon Cognito, JWT Authentication, API Gateway Authorizer, kiến trúc Serverless, xử lý hướng sự kiện, Data Analytics và các dịch vụ AI/ML trên AWS.

**4. Cơ hội học hỏi và phát triển kỹ năng**  
Trong chương trình, em đã thực hiện các bài lab theo tuần liên quan đến Amazon EC2, Amazon S3, Amazon RDS, Amazon VPC, AWS IAM, Amazon CloudWatch, AWS Tags, Resource Groups, Data Analytics và AI/ML Services. Các bài thực hành giúp em hiểu rõ hơn cách nhiều dịch vụ AWS có thể kết hợp với nhau để xây dựng một hệ thống hoàn chỉnh, thay vì chỉ sử dụng từng dịch vụ riêng lẻ.

Trong đồ án nhóm, em đảm nhận vai trò **Authentication & API Developer** cho dự án Secure AI-Driven Document Platform. Em đã cấu hình Amazon Cognito User Pool, tạo App Client dành cho Single Page Application, sử dụng Authorization Code Flow kết hợp PKCE, tạo HTTP API trên Amazon API Gateway, cấu hình JWT Authorizer, xây dựng các custom OAuth scope và tạo hai nhóm `users`, `admins`. Em cũng thực hiện kiểm thử các trường hợp không có token, token không hợp lệ, người dùng hợp lệ, người dùng không có quyền quản trị và tài khoản quản trị. Đây là phần công việc giúp em phát triển kỹ năng kỹ thuật nhiều nhất trong kỳ thực tập.

**5. Kỹ năng làm việc nhóm và tác phong chuyên nghiệp**  
Đồ án nhóm giúp em hiểu rõ hơn tầm quan trọng của việc phân chia trách nhiệm và thống nhất giao diện kỹ thuật giữa các module. Thành phần Authentication và API do em triển khai phải tương thích với frontend và các Lambda do thành viên backend xây dựng. Vì vậy, em học được cách trao đổi rõ ràng về API route, vị trí JWT claims, scope, group, mã trạng thái HTTP và định dạng request/response. Thông qua seminar, báo cáo tuần và quá trình thảo luận đồ án, em cũng cải thiện kỹ năng trình bày, viết tài liệu, quản lý thời gian và phối hợp nhóm.

**6. Điều em hài lòng nhất trong kỳ thực tập**  
Điều khiến em hài lòng nhất là hoàn thành được luồng xác thực và phân quyền từ đầu đến cuối. Người dùng có thể đăng nhập thông qua Amazon Cognito, nhận Access Token, gọi API được bảo vệ và để Amazon API Gateway kiểm tra JWT trước khi chuyển request đến AWS Lambda. Kết quả kiểm thử thể hiện rõ: request không có token hoặc sử dụng token sai trả về `401`; người dùng thông thường gọi API quản trị trả về `403`; tài khoản thuộc nhóm `admins` nhận phản hồi thành công `200`. Qua kết quả thực tế này, em hiểu rõ hơn sự khác biệt giữa Authentication và Authorization.

---

### Đề xuất cải thiện chương trình

Theo em, chương trình có thể cung cấp lộ trình đồ án rõ ràng hơn ngay từ giai đoạn đầu, bao gồm kiến trúc dự kiến, vai trò của từng thành viên, quy ước đặt tên tài nguyên và các mốc tích hợp. Điều này sẽ giúp các thành viên phối hợp tốt hơn và hạn chế việc phải điều chỉnh lại khi mỗi người có một cách hiểu kỹ thuật khác nhau.

Ngoài ra, chương trình có thể tổ chức thêm các buổi review kỹ thuật ngắn trong thời gian thực hiện đồ án. Mỗi nhóm có thể trình bày kiến trúc hiện tại, các module đã hoàn thành, vấn đề tích hợp và kế hoạch tiếp theo. Việc review sớm sẽ giúp phát hiện các vấn đề liên quan đến API contract, quyền truy cập, lựa chọn dịch vụ và luồng xử lý trước giai đoạn tích hợp cuối cùng.

Do chương trình được tổ chức theo hình thức hybrid, em cũng mong muốn có thêm một số hoạt động giao lưu hoặc chia sẻ chéo giữa các nhóm. Điều này sẽ tạo cơ hội để sinh viên trao đổi kinh nghiệm, học hỏi cách triển khai của nhóm khác và cải thiện kỹ năng giao tiếp.

---

### Đánh giá và mong muốn trong tương lai

Em sẽ giới thiệu chương trình First Cloud AI Journey cho những sinh viên muốn tìm hiểu điện toán đám mây thông qua các hoạt động thực hành. Chương trình cung cấp kiến thức về nhiều dịch vụ AWS và tạo cơ hội cho sinh viên xây dựng một dự án kết hợp hạ tầng Cloud, bảo mật, API, dữ liệu và AI. Tuy nhiên, người tham gia cần chủ động đọc tài liệu, lưu lại tiến độ và thực hành thường xuyên vì lượng kiến thức mới tương đối lớn.

Trong tương lai, em mong muốn tiếp tục học và thực hành AWS, đặc biệt trong các lĩnh vực phát triển backend Serverless, bảo mật Cloud, Infrastructure as Code, giám sát hệ thống và CI/CD. Em cũng mong muốn tiếp tục hoàn thiện đồ án bằng cách tích hợp đầy đủ các API nghiệp vụ, tăng cường kiểm tra quyền sở hữu tài liệu, cải thiện logging, monitoring và tự động hóa quá trình triển khai.

Nhìn chung, kỳ thực tập đã giúp em có góc nhìn thực tế hơn về cách xây dựng hệ thống trên nền tảng Cloud, đồng thời giúp em xác định rõ những kiến thức và kỹ năng cần tiếp tục phát triển để theo đuổi định hướng Backend Developer và Cloud Developer trong tương lai.