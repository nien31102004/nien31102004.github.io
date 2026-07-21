---
title : "Xem kết quả quét tài liệu"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.8.5 </b> "
---

Trong bước này, chúng ta sẽ sử dụng Lambda `GetDocumentScanResultLambda` để trả về toàn bộ trạng thái quét và đánh giá của một tài liệu.

Lambda được tích hợp với route:

```text
GET /documents/{documentId}/scan-result
```

Biến môi trường:

```text
DOCUMENTS_TABLE = SecureDocuments
```

Lambda nhận `documentId`, đọc item từ bảng `SecureDocuments` và kiểm tra thuộc tính `userId` có trùng với claim `sub` trong JWT Token hay không.

Execution role của Lambda được cấp quyền:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadDocumentScanResult",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:950725740411:table/SecureDocuments"
    }
  ]
}
```

Response được chia thành các nhóm thông tin:

```text
processing
malwareScan
aiAnalysis
systemDecision
review
finalResult
```

Phần `processing` cho biết giai đoạn xử lý hiện tại. Lambda ánh xạ trạng thái tài liệu như sau:

```text
PENDING_UPLOAD         → UPLOAD
WAITING_MALWARE_SCAN   → MALWARE_SCAN
AI_ANALYZING           → AI_ANALYSIS
AI_ANALYSIS_COMPLETED  → DECISION
MOVING_TO_CLEAN        → FINALIZING
MOVING_TO_REJECT       → FINALIZING
MANUAL_REVIEW          → MANUAL_REVIEW
SAFE                   → COMPLETED
REJECTED               → COMPLETED
AI_ERROR               → ERROR
SCAN_ERROR             → ERROR
```

Phần `malwareScan` trả về:

```text
status
result
threats
statusReasons
scannedAt
```

Phần `aiAnalysis` trả về:

```text
status
provider
model
riskScore
riskLevel
recommendedAction
confidence
summary
categories
phishingDetected
socialEngineeringDetected
credentialTheftDetected
promptInjectionDetected
suspiciousLinks
completedAt
```

Phần `systemDecision` trả về điểm rủi ro, mức độ rủi ro, hành động đề xuất và lý do của hệ thống.

Phần `review` trả về kết quả đánh giá thủ công của quản trị viên.

Phần `finalResult` trả về quyết định cuối cùng và xác định tài liệu có được phép tải xuống hay không:

```python
downloadAvailable = (
    item.get("status") == "SAFE"
    and item.get("finalDecision") == "ALLOW"
    and item.get("currentPrefix") == "clean"
)
```

Gửi yêu cầu:

```http
GET https://u0figy4o19.execute-api.ap-southeast-1.amazonaws.com/documents/<DOCUMENT_ID>/scan-result
Authorization: Bearer <ACCESS_TOKEN>
```

Response thành công có dạng:

```json
{
  "documentId": "e509813c-52b4-4558-ace0-d07c18fbf74d",
  "fileName": "admin.txt",
  "contentType": "text/plain",
  "fileSize": 84,
  "status": "SAFE",
  "processing": {
    "stage": "COMPLETED",
    "complete": true
  },
  "malwareScan": {
    "status": "COMPLETED",
    "result": "NO_THREATS_FOUND",
    "threats": [],
    "statusReasons": [],
    "scannedAt": "2026-07-19T08:02:03Z"
  },
  "aiAnalysis": {
    "status": "COMPLETED",
    "provider": "BEDROCK_MANTLE",
    "model": "google.gemma-3-4b-it",
    "riskScore": 75,
    "riskLevel": "HIGH",
    "recommendedAction": "REJECT",
    "confidence": 0.95,
    "summary": "The document contains a suspicious URL...",
    "categories": [
      "Phishing",
      "Credential Theft",
      "Impersonation"
    ],
    "phishingDetected": true,
    "socialEngineeringDetected": true,
    "credentialTheftDetected": true,
    "promptInjectionDetected": false,
    "suspiciousLinks": [],
    "completedAt": "2026-07-19T08:02:08.098114+00:00"
  },
  "systemDecision": {
    "riskScore": 75,
    "riskLevel": "HIGH",
    "recommendedAction": "REVIEW",
    "reason": "Administrator review required...",
    "decisionOverridden": true
  },
  "review": {
    "status": "APPROVE",
    "adminDecision": "APPROVE",
    "adminRiskScore": 0,
    "adminReason": "Không có nội dung nguy hiểm",
    "reviewedAt": "2026-07-19T08:48:48.433094+00:00"
  },
  "finalResult": {
    "decision": "ALLOW",
    "decisionSource": "ADMIN_REVIEW",
    "riskScore": 75,
    "riskLevel": "HIGH",
    "downloadAvailable": true
  }
}
```

![Get scan result successfully](/images/5-Workshop/5.8/5.8.5/5.8.5.1.png)

{{% notice note %}}
Danh sách `threats`, `statusReasons`, `categories` và `suspiciousLinks` được giới hạn tối đa 20 phần tử. Mỗi chuỗi được giới hạn tối đa 500 ký tự trước khi trả về client.
{{% /notice %}}
