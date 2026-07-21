---
title : "Kiểm tra phân tích tài liệu bằng AI"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

Để kiểm tra quá trình phân tích AI, tải một tài liệu an toàn lên hệ thống bằng Presigned URL.

Có thể sử dụng tài liệu PDF, TXT hoặc DOCX chứa nội dung thông thường như báo cáo dự án, tài liệu kỹ thuật hoặc hướng dẫn sử dụng AWS.

![Upload test document](/images/5-Workshop/5.6/5.6.4/5.6.4.1.png)

Sau khi tải lên, đợi Amazon GuardDuty hoàn thành quá trình quét mã độc.

Khi GuardDuty trả về:

```text
NO_THREATS_FOUND
```

Lambda `SecureDocScanOrchestrator` sẽ tự động được kích hoạt thông qua Amazon SQS.

Mở [Amazon CloudWatch console](https://console.aws.amazon.com/cloudwatch/), chọn **Log groups** và mở log group:

```text
message": "Invoking Bedrock Mantle Chat Completions",
"endpoint": "https://bedrock-mantle.ap-northeast-1.api.aws/v1/chat/completions",
"region": "ap-northeast-1",
"model": "google.gemma-3-4b-it",
"documentId": "df461ff5-6daa-4c3b-a2dd-e32515c30359",
"characterCount": 570
```

![Open Lambda logs](/images/5-Workshop/5.6/5.6.4/5.6.4.2.png)

Kiểm tra log trích xuất nội dung tài liệu:

```text
Document text extraction completed
```

Log sẽ hiển thị các thông tin như:

+ Loại file được phát hiện.
+ Số ký tự được trích xuất.
+ Nội dung có bị cắt bớt hay không.
+ Số trang PDF.
+ Số đoạn văn và bảng trong DOCX.
+ Phương pháp trích xuất.

![Extraction log](/images/5-Workshop/5.6/5.6.4/5.6.4.3.png)

Tiếp tục kiểm tra log gọi Bedrock Mantle:

```text
Invoking Bedrock Mantle Chat Completions
```

Khi phân tích thành công, log hiển thị:

```text
AI analysis completed
```

![AI completed log](/images/5-Workshop/5.6/5.6.4/5.6.4.4.png)

Mở Amazon DynamoDB, chọn bảng:

```text
SecureDocuments
```

Tìm bản ghi theo `documentId` và kiểm tra các thuộc tính:

+ `aiStatus`: `COMPLETED`
+ `aiProvider`: `BEDROCK_MANTLE`
+ `riskScore`
+ `riskLevel`
+ `aiSummary`
+ `aiConfidence`
+ `aiRecommendedAction`
+ `systemRecommendedAction`
+ `detectedFileType`
+ `extractionMetadata`
+ `decisionStatus`

![DynamoDB AI result](/images/5-Workshop/5.6/5.6.4/5.6.4.5.png)

Với tài liệu an toàn, kết quả dự kiến:

```json
{
  "aiStatus": "COMPLETED",
  "riskLevel": "LOW",
  "systemRecommendedAction": "ALLOW",
  "decisionStatus": "READY_FOR_EXECUTION"
}
```

Sau khi lưu kết quả thành công, Lambda sẽ gọi bất đồng bộ hàm:

```text
SecureDocDecisionEngine
```

Decision Engine tiếp tục chuyển tài liệu đến vùng lưu trữ phù hợp.

{{% notice note %}}
Tài liệu có điểm rủi ro từ `0` đến `20`, độ tin cậy đạt yêu cầu và không có tín hiệu nguy hiểm sẽ được đề xuất `ALLOW`.
{{% /notice %}}

{{% notice note %}}
Tài liệu có kết quả không rõ ràng, độ tin cậy thấp hoặc có sự khác biệt giữa mô hình AI và quy tắc cục bộ sẽ được chuyển sang `REVIEW` để quản trị viên đánh giá.
{{% /notice %}}

{{% notice warning %}}
Nếu xuất hiện lỗi thiếu thư viện `pypdf` hoặc `python-docx`, hãy kiểm tra lại file ZIP. Nếu file TXT không thể giải mã, hãy lưu lại tài liệu bằng UTF-8 hoặc một encoding được Lambda hỗ trợ.
{{% /notice %}}