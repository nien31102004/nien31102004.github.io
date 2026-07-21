---
title : "Chuyển tài liệu an toàn"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2 </b> "
---

Khi kết quả cuối cùng là `ALLOW`, Decision Engine chuyển tài liệu từ prefix `quarantine/` sang prefix `clean/`.

Tài liệu chỉ được phép chuyển sang vùng an toàn khi:

+ `malwareStatus` là `NO_THREATS_FOUND`.
+ `aiStatus` là `COMPLETED`.
+ `systemRecommendedAction` là `ALLOW`, hoặc quản trị viên chấp thuận tài liệu đang chờ đánh giá.

Luồng di chuyển được thực hiện như sau:

```text
quarantine/{userId}/{documentId}/{fileName}
```

được chuyển thành:

```text
clean/{userId}/{documentId}/{fileName}
```

Decision Engine cập nhật trạng thái tạm thời của tài liệu thành:

```text
MOVING_TO_CLEAN
```

![Moving to clean](/images/5-Workshop/5.7/5.7.2/5.7.2.1.png)

Lambda sử dụng `CopyObject` để sao chép tài liệu đến prefix `clean/`.

Sau khi sao chép, Lambda kiểm tra:

+ Object đích có tồn tại.
+ ETag của object đích hợp lệ.
+ Kích thước object đích bằng object nguồn.
+ Version ID được lưu lại nếu S3 Versioning đang bật.

Khi xác minh thành công, bản ghi DynamoDB được cập nhật:

+ `status`: `SAFE`
+ `finalDecision`: `ALLOW`
+ `currentPrefix`: `clean`
+ `s3Key`: đường dẫn mới trong `clean/`
+ `finalDecisionSource`: `SYSTEM_RULES` hoặc `ADMIN_REVIEW`
+ `sourceCleanupStatus`: `PENDING`

![Safe document record](/images/5-Workshop/5.7/5.7.2/5.7.2.2.png)

Sau đó Lambda xóa object cũ trong prefix `quarantine/` và cập nhật:

```text
sourceCleanupStatus = COMPLETED
```

![Clean object](/images/5-Workshop/5.7/5.7.2/5.7.2.3.png)

{{% notice note %}}
Quá trình di chuyển trong Amazon S3 thực chất gồm hai bước: sao chép object sang đường dẫn mới và xóa object ở đường dẫn cũ.
{{% /notice %}}

{{% notice warning %}}
Decision Engine xác minh object đích trước khi xóa object nguồn. Cơ chế này giúp hạn chế mất tài liệu nếu thao tác sao chép gặp lỗi.
{{% /notice %}}