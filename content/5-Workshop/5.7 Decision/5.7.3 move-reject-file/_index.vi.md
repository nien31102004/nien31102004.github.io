---
title : "Chuyển tài liệu bị từ chối"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3 </b> "
---

Khi phát hiện mã độc hoặc nội dung có mức độ rủi ro cao, Decision Engine chuyển tài liệu từ prefix `quarantine/` sang prefix `reject/`.

Tài liệu bị từ chối khi:

+ GuardDuty trả về `THREATS_FOUND`.
+ Hệ thống AI đề xuất `REJECT` và đáp ứng các quy tắc xác nhận rủi ro.
+ Quản trị viên chọn từ chối tài liệu đang chờ đánh giá.

Đường dẫn tài liệu được chuyển từ:

```text
quarantine/{userId}/{documentId}/{fileName}
```

sang:

```text
reject/{userId}/{documentId}/{fileName}
```

Decision Engine cập nhật trạng thái tạm thời:

```text
MOVING_TO_REJECT
```

Lambda sao chép tài liệu sang prefix `reject/` và kiểm tra object đích trước khi cập nhật trạng thái cuối cùng.

Sau khi hoàn thành, DynamoDB được cập nhật:

+ `status`: `REJECTED`
+ `finalDecision`: `REJECT`
+ `currentPrefix`: `reject`
+ `s3Key`: đường dẫn mới trong `reject/`
+ `finalRiskScore`: điểm rủi ro cuối cùng
+ `finalRiskLevel`: mức độ rủi ro cuối cùng
+ `finalDecisionSource`: nguồn đưa ra quyết định

![Rejected document record](/images/5-Workshop/5.7/5.7.3/5.7.3.1.png)

Object nguồn trong prefix `quarantine/` được xóa sau khi object đích đã được xác minh.


Nếu kết quả hệ thống là `REVIEW`, tài liệu không được di chuyển mà tiếp tục nằm trong prefix `quarantine/` với các trạng thái:

```text
status = MANUAL_REVIEW
reviewStatus = PENDING
finalDecision = PENDING
```

{{% notice warning %}}
Nếu GuardDuty phát hiện `THREATS_FOUND`, kết quả AI không được phép ghi đè quyết định. Tài liệu luôn được xử lý theo hướng `REJECT`.
{{% /notice %}}

{{% notice note %}}
Tài liệu ở trạng thái `MANUAL_REVIEW` vẫn nằm trong vùng `quarantine/` cho đến khi quản trị viên chọn `ALLOW` hoặc `REJECT`.
{{% /notice %}}