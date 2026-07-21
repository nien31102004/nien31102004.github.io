---
title: "Worklog Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

* Tổng hợp tiến độ dự án Secure AI-Driven Document Platform và rà soát kiến trúc Serverless, Event-Driven.
* Hoàn thiện tài liệu cho phần Authentication & API do em phụ trách.
* Chuẩn hóa luồng đăng nhập, JWT Authentication, phân quyền User/Admin và các API tài liệu.
* Chuẩn bị hình ảnh, test case và kịch bản demo cuối kỳ.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Rà soát kiến trúc tổng thể; thống nhất sử dụng HTTP API, kiểm tra tên Cognito, API Gateway, Lambda, S3, SQS và DynamoDB trên sơ đồ. | 29/06/2026 | 29/06/2026 | <https://docs.aws.amazon.com/whitepapers/latest/serverless-multi-tier-architectures-api-gateway-lambda/> |
| 3 | Tổng hợp cấu hình Cognito User Pool, SPA App Client, Managed Login, Authorization Code Flow with PKCE và callback URL. | 30/06/2026 | 30/06/2026 | <https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-managed-login.html> |
| 4 | Chuẩn hóa JWT Authorizer của HTTP API; mô tả Issuer, Audience, `Authorization: Bearer`, claim `sub` và route kiểm thử `/me`. | 01/07/2026 | 01/07/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-jwt-authorizer.html> |
| 5 | Hoàn thiện Resource Server, scopes `upload/read/admin`, Cognito groups `users/admins` và kiểm thử phân quyền hai lớp. | 02/07/2026 | 02/07/2026 | <https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-define-resource-servers.html> |
| 6 | Phối hợp với Backend để thống nhất route tài liệu, cách đọc JWT claims, sử dụng `sub` làm owner ID và kiểm tra quyền sở hữu tài liệu. | 03/07/2026 | 03/07/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html> |
| 7 và CN | Sắp xếp ảnh minh chứng `401/403/200`, hoàn thiện test matrix, kịch bản demo và rà soát báo cáo dự án. | 04/07/2026 | 05/07/2026 | <https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/> |

### Kết quả đạt được tuần 11:

* Chuẩn hóa kiến trúc dự án theo HTTP API và luồng xử lý Serverless, Event-Driven.
* Hoàn thiện tài liệu Authentication & API gồm Cognito, OAuth 2.0 với PKCE, JWT Authorizer, scopes và groups.
* Tổng hợp được các kết quả kiểm thử: không token `401`, token giả `401`, token hợp lệ `200`, user gọi API admin `403`, admin gọi API admin `200`.
* Thống nhất với Backend cách lấy `sub` từ JWT, không nhận `userId` từ frontend và kiểm tra ownership tài liệu.
* Hoàn thiện bộ ảnh minh chứng, test case và kịch bản demo để sử dụng trong tuần cuối.

### Đánh giá cuối tuần:

* Các mục tiêu chính của tuần đã được hoàn thành theo kế hoạch.
* Nội dung thực hành giúp em củng cố kiến thức lý thuyết và rèn luyện khả năng triển khai trên AWS.
* Các khó khăn phát sinh được ghi nhận, kiểm tra và xử lý thông qua tài liệu AWS cùng quá trình trao đổi với mentor và thành viên nhóm.
* Em đã tổng hợp kết quả, rà soát tài nguyên và lưu lại các nội dung cần thiết để phục vụ báo cáo thực tập.
