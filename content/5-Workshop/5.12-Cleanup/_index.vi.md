---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

Xóa tài nguyên workshop sau khi kiểm thử để tránh chi phí ngoài dự kiến và loại bỏ dữ liệu thử nghiệm, thông tin xác thực cùng endpoint công khai.

{{% notice danger %}}
Cleanup có tính phá hủy và thường không thể hoàn tác. Trước khi xóa, hãy xác minh AWS account ID, mọi Region đang dùng, tên tài nguyên và tài nguyên có được workload khác chia sẻ hay không. Không áp dụng nguyên checklist này cho tài khoản production.
{{% /notice %}}

Thực hiện đúng thứ tự bên dưới. Xóa IAM role, bucket hoặc table quá sớm có thể khiến dịch vụ không hoàn tất xử lý hoặc làm tài nguyên phụ thuộc khó xóa hơn.

## 1. Lưu bằng chứng và xác minh phạm vi

Lưu screenshot, kết quả kiểm thử, bản xuất cấu hình hoặc log cần cho báo cáo workshop. Không xuất access token, presigned URL, mật khẩu, API key hoặc nội dung tài liệu.

Xác nhận danh tính hiện tại và Region mặc định:

```bash
aws sts get-caller-identity
aws configure get region
```

Kiểm tra ít nhất:

```text
ap-southeast-1   Tài nguyên workshop chính
ap-northeast-1   Bedrock Mantle API key dùng ở phần 5.6
Global           CloudFront và IAM
```

Lập danh sách trước khi xóa:

| Dịch vụ | Tài nguyên workshop |
|---|---|
| CloudFront | Frontend distribution và OAC |
| S3 | Bucket quarantine, clean, rejected và frontend |
| Cognito | User pool, app client, domain, group và tài khoản thử nghiệm |
| API Gateway | `secure-document-api` |
| Lambda | Hàm upload, malware result, AI, decision, document và Admin; layer của workshop nếu có |
| DynamoDB | `secure-documents`, stream, index và manual backup tùy chọn |
| GuardDuty | Malware Protection plan cho bucket quarantine |
| SQS | `secure-doc-upload-events-demo` và dead-letter queue nếu có |
| EventBridge | Rule kết quả malware gọi `secure-doc-process-scan-result` |
| Secrets Manager | `secure-doc/mantle-api-key` |
| IAM | Lambda execution role, GuardDuty role và policy workshop |
| CloudWatch | Lambda/API log group, alarm và dashboard |

Nếu có gắn tag, dùng Tag Editor hoặc Resource Groups Tagging API làm nguồn kiểm kê bổ sung. Tag không bao phủ mọi loại tài nguyên nên vẫn phải kiểm tra từng service console.

## 2. Ngăn yêu cầu mới

Hoàn tất hoặc hủy mọi bài kiểm thử đang chạy và đóng frontend.

1. Disable CloudFront distribution để người dùng không mở được ứng dụng.
2. Xóa `secure-document-api`, hoặc trước tiên gỡ các route nếu cần giữ bằng chứng thêm một thời gian ngắn.
3. Xác nhận không còn client workshop tải trực tiếp bằng presigned URL chưa hết hạn.
4. Chờ Lambda invocation và xử lý bất đồng bộ đang hoạt động dừng hoàn toàn.

Không xóa DynamoDB hoặc bucket dữ liệu khi pipeline vẫn đang xử lý object.

## 3. Tắt nguồn sự kiện và quét malware

Dừng nguồn tạo sự kiện bất đồng bộ trước khi xóa consumer:

1. Trong từng Lambda, disable hoặc xóa DynamoDB event source mapping của `SecureDocAIAnalysis` và `SecureDocDecisionEngine`.
2. Gỡ S3 event notification `secure-doc-upload-created-demo` khỏi bucket quarantine.
3. Trong EventBridge, gỡ target của rule kết quả malware rồi xóa rule.
4. [Disable GuardDuty Malware Protection plan](https://docs.aws.amazon.com/guardduty/latest/ug/disable-malware-s3-protected-bucket.html) cho `securedocs-quarantine-<account-id>`.
5. Xác nhận `secure-doc-upload-events-demo` và dead-letter queue không còn nhận message.
6. Purge hoặc xóa các queue workshop sau khi xác nhận không còn cần bằng chứng.

Tắt protection plan của bucket không bắt buộc tắt toàn bộ dịch vụ GuardDuty. Nếu GuardDuty đã được bật cho tài khoản hoặc bảo vệ workload khác, hãy giữ nguyên detector và các protection plan khác.

Chờ protection plan bị gỡ trước khi xóa GuardDuty IAM role hoặc bucket quarantine.

## 4. Xóa CloudFront an toàn

[CloudFront yêu cầu distribution phải bị disable trước khi xóa](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowToDeleteDistribution.html).

1. Mở CloudFront distribution và chọn **Disable**.
2. Chờ distribution cập nhật có trạng thái **Deployed** và được hiển thị là disabled.
3. Xóa distribution.
4. Xóa Origin Access Control của workshop sau khi nó không còn gắn với distribution.
5. Gỡ statement CloudFront khỏi bucket policy của bucket frontend.

Nếu đã cấu hình custom domain, gỡ Route 53 alias record. Chỉ xóa ACM certificate khi không có distribution hoặc dịch vụ khác sử dụng. Thay đổi CloudFront là toàn cục và cần thời gian lan truyền; không giả định thao tác xóa hoàn tất ngay lập tức.

## 5. Xóa Lambda và API

Sau khi mọi trigger và API integration đã ngừng hoạt động, xóa các hàm workshop gồm:

```text
secure-doc-generate-upload-url
secure-doc-process-scan-result
SecureDocAIAnalysis
SecureDocDecisionEngine
ListMyDocumentsLambda
GetMyDocumentLambda
DeleteMyDocumentLambda
GenerateDownloadUrlLambda
GetDocumentScanResultLambda
AdminListDocumentsFunction
AdminGetDocumentFunction
AdminReviewDocumentFunction
AdminDeleteDocumentFunction
AdminManageUsersFunction
```

Xóa Lambda layer workshop đã tạo cho thư viện trích xuất tài liệu nếu có. Tên hàm có thể khác nếu bạn đổi trong workshop, vì vậy hãy so sánh Lambda console với danh sách đã kiểm kê thay vì chỉ dựa vào danh sách trên.

Tạm thời giữ CloudWatch log group nếu vẫn cần bằng chứng; log group không nhất thiết bị xóa cùng Lambda.

## 6. Thu hồi thông tin AI và xóa secret

Trong Amazon Bedrock console tại `ap-northeast-1`, thu hồi hoặc xóa Mantle API key đã tạo cho workshop. Thu hồi credential ở nguồn trước giúp secret bị sao chép cũng không thể tiếp tục gọi dịch vụ.

Sau đó xóa Secrets Manager secret:

```text
secure-doc/mantle-api-key
```

Dùng recovery window nếu chính sách tổ chức yêu cầu. Chỉ force delete không khôi phục khi đã xác nhận secret chỉ thuộc workshop và chính sách cho phép. Kiểm tra replica ở Region khác và xóa nếu có.

## 7. Xóa tài nguyên Cognito

Nếu user pool chỉ được tạo cho workshop:

1. Xóa Cognito managed-login domain.
2. Xóa app client của workshop.
3. Xóa group thử nghiệm `Users`, `Admins` hoặc chuyển thẳng sang xóa user pool.
4. Xóa user pool cùng mọi tài khoản thử nghiệm.

Nếu user pool được chia sẻ, không xóa toàn bộ pool. Chỉ gỡ app client, callback/sign-out URL, domain workshop nếu phù hợp, tài khoản thử nghiệm và group membership do dự án tạo.

Xóa frontend hoặc CloudFront không làm JWT đã cấp mất hiệu lực ngay lập tức. Việc gỡ API và app client ở bước trước bảo đảm token không tiếp tục gọi backend workshop.

## 8. Xóa bảng DynamoDB

Xác nhận Lambda event source mapping đã biến mất. [Tắt DynamoDB Stream](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html) nếu tạm thời giữ table; nếu không, xóa:

```text
secure-documents
```

Xóa table cũng xóa các global secondary index. Tắt deletion protection trước nếu đã bật.

Kiểm tra riêng mục **Backups**. On-demand backup có thể vẫn tồn tại sau khi table bị xóa và tiếp tục phát sinh chi phí lưu trữ. Chỉ xóa backup workshop không còn cần cho bằng chứng hoặc lưu giữ.

## 9. Làm trống và xóa mọi bucket S3

Xóa các bucket workshop:

```text
securedocs-quarantine-<account-id>
securedocs-clean-<account-id>
securedocs-rejected-<account-id>
securedocs-frontend-<account-id>
```

[Mọi object version và delete marker phải bị xóa trước khi có thể xóa bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html). Với từng bucket:

1. Gỡ event notification, replication configuration, access point và bucket policy workshop còn lại nếu có.
2. Chọn **Empty** trong S3 console và xác nhận xóa vĩnh viễn.
3. Khi versioning đã bật hoặc suspended, xác minh **mọi object version và mọi delete marker** đều bị xóa.
4. Xóa bucket đã trống.

{{% notice warning %}}
`aws s3 rm --recursive` xóa object hiện tại nhưng tự nó chưa đủ cho bucket có versioning; lệnh có thể để lại noncurrent version và delete marker. Không thể xóa bucket đến khi mọi version và delete marker biến mất.
{{% /notice %}}

Dùng kiểm tra chỉ đọc trước khi xóa bucket:

```bash
aws s3api list-object-versions \
  --bucket securedocs-quarantine-<account-id>
```

Response không được còn `Versions` hoặc `DeleteMarkers`. Lặp lại với cả bốn bucket.

## 10. Xóa IAM, log và tài nguyên hỗ trợ

Sau khi dịch vụ sử dụng chúng đã bị xóa:

1. Xóa inline policy workshop và detach managed policy khỏi Lambda execution role.
2. Xóa Lambda execution role và GuardDuty Malware Protection role.
3. Xóa CloudWatch log group, alarm, dashboard và saved query của workshop sau khi xuất bằng chứng cần thiết.
4. Xóa EventBridge archive, SQS dead-letter queue, SNS topic hoặc CloudWatch Logs resource policy riêng của workshop nếu có.
5. Chỉ xóa customer-managed KMS key nếu nó được tạo riêng cho workshop và không có dữ liệu mã hóa hoặc dịch vụ khác sử dụng. Xóa KMS key là thao tác lên lịch và có thời gian chờ.
6. Gỡ Route 53 record, custom-domain resource, WAF web ACL association và ACM certificate chỉ khi không được chia sẻ.

Không xóa CloudTrail cấp tài khoản, GuardDuty detector, AWS Config, dịch vụ bảo mật, service-linked role hoặc tài nguyên mạng dùng chung chỉ vì workshop đã sử dụng chúng.

## 11. Xác minh cleanup

Dùng service console và các lệnh chỉ đọc để tìm tài nguyên còn sót:

```bash
aws s3api list-buckets
aws lambda list-functions --region ap-southeast-1
aws dynamodb list-tables --region ap-southeast-1
aws apigatewayv2 get-apis --region ap-southeast-1
aws sqs list-queues --region ap-southeast-1
aws secretsmanager list-secrets --region ap-southeast-1
aws cloudfront list-distributions
```

Đồng thời xác nhận:

- Không có Malware Protection plan tham chiếu bucket quarantine.
- Không có Lambda event source mapping tham chiếu DynamoDB Stream hoặc queue đã xóa.
- Không có EventBridge rule gọi Lambda đã xóa.
- Không có CloudFront distribution hoặc OAC tham chiếu bucket frontend.
- Không vô tình để lại manual DynamoDB backup, S3 version, delete marker, Secrets Manager replica hoặc log group.
- Cognito managed-login domain và CloudFront URL không còn phục vụ ứng dụng.

Dữ liệu Billing và Cost Explorer có thể cập nhật chậm. Kiểm tra lại chi phí dịch vụ sau khi dữ liệu báo cáo cập nhật và xác nhận không còn tài nguyên workshop tiếp tục tạo usage.

## Checklist cleanup

| Nhóm tài nguyên | Hoàn tất |
|---|---|
| Đã lưu bằng chứng và xác minh account/Region | ☐ |
| Đã dừng truy cập Frontend và API | ☐ |
| Đã gỡ Lambda trigger và EventBridge rule | ☐ |
| Đã tắt GuardDuty Malware Protection plan | ☐ |
| Đã xóa CloudFront distribution và OAC | ☐ |
| Đã xóa Lambda, layer, API và queue | ☐ |
| Đã thu hồi Mantle API key và xóa secret | ☐ |
| Đã xóa tài nguyên Cognito của workshop | ☐ |
| Đã xóa DynamoDB table và backup không cần thiết | ☐ |
| Đã xóa mọi S3 version, delete marker và bucket | ☐ |
| Đã xóa IAM role, log, alarm và tài nguyên tùy chọn | ☐ |
| Đã kiểm kê cuối và kiểm tra Billing | ☐ |

Cleanup hoàn tất khi danh sách cuối không còn tài nguyên workshop ngoài ý muốn và mọi backup, log, dịch vụ bảo mật dùng chung, certificate hoặc domain được giữ lại đều có chủ sở hữu cùng lý do lưu giữ rõ ràng.
