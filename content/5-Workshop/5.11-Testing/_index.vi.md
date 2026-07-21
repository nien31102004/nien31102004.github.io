---
title : "Kiểm thử"
date : 2024-01-01
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

Phần này xác minh toàn bộ luồng tài liệu an toàn, từ xác thực và tải lên đến quét mã độc, phân tích AI, quyết định cuối, quản trị và phân phối qua CloudFront.

Mục tiêu không chỉ là xác nhận request thành công. Kiểm thử phải chứng minh tài liệu không an toàn không thể tải xuống, người dùng không thể truy cập tài liệu của nhau, sự kiện lặp không tạo lần di chuyển trùng và điều khiển frontend luôn được bảo vệ thêm bằng phân quyền phía server.

## Chuẩn bị môi trường kiểm thử

Tạo ba tài khoản thử nghiệm trong Cognito user pool:

| Tài khoản | Nhóm | Mục đích |
|---|---|---|
| `workshop-user-a` | `Users` | Luồng người dùng chính |
| `workshop-user-b` | `Users` | Kiểm tra cô lập quyền sở hữu |
| `workshop-admin` | `Admins` | Kiểm duyệt và quản lý người dùng |

Chuẩn bị các tệp thử nghiệm không dùng trong production:

- Một tệp PDF hoặc văn bản thông thường, kích thước nhỏ, dự kiến an toàn.
- Một tài liệu vô hại chứa nội dung rủi ro cao hoặc giống phishing có kiểm soát để kiểm tra AI.
- Một tài liệu không rõ ràng dự kiến cần kiểm duyệt thủ công.
- Tệp kiểm thử chống mã độc EICAR tiêu chuẩn đã dùng ở phần 5.5. EICAR là chữ ký kiểm thử, không phải mã độc thật, nhưng chỉ sử dụng trong tài khoản workshop cô lập.

Ghi lại giá trị tài nguyên hiện tại:

```text
CloudFront URL
API Gateway invoke URL
Cognito User Pool và App Client ID
Bảng secure-documents
securedocs-quarantine-<account-id>
securedocs-clean-<account-id>
securedocs-rejected-<account-id>
```

Trước khi kiểm thử, xác nhận:

- Cognito callback và sign-out URL có CloudFront domain.
- API Gateway JWT authorization và CORS đã bật cho mọi route được bảo vệ.
- S3 CORS cho phép CloudFront origin tải lên với các header bắt buộc.
- GuardDuty, DynamoDB Streams, S3 notification và Lambda trigger đã bật.
- `SecureDocDecisionEngine` lọc `MALWARE_DETECTED`, `AI_ANALYSIS_COMPLETED` và `ADMIN_DECISION_READY`.
- CloudFront đã deploy, S3 origin dùng OAC và truy cập trực tiếp bucket frontend bị từ chối.

Dùng tên tệp riêng cho mỗi lần chạy, ví dụ `T02-safe-20260721.pdf`. Ghi lại từng `documentId` được trả về; không dùng lại tài liệu cũ để kiểm tra nhánh khác.

## Chuyển đổi trạng thái mong đợi

| Kịch bản | Chuyển đổi mong đợi |
|---|---|
| Tài liệu an toàn | `PENDING_UPLOAD → MALWARE_CLEAN → AI_ANALYZING → AI_ANALYSIS_COMPLETED → DECISION_PROCESSING → SAFE` |
| Phát hiện malware | `PENDING_UPLOAD → MALWARE_DETECTED → DECISION_PROCESSING → REJECTED` |
| Kiểm duyệt thủ công | `PENDING_UPLOAD → MALWARE_CLEAN → AI_ANALYZING → AI_ANALYSIS_COMPLETED → DECISION_PROCESSING → MANUAL_REVIEW` |
| Quyết định Admin | `MANUAL_REVIEW → ADMIN_DECISION_READY → DECISION_PROCESSING → SAFE hoặc REJECTED` |
| Lỗi xử lý | `MALWARE_SCAN_ERROR`, `AI_ERROR` hoặc `DECISION_ERROR` |

Một số trạng thái trung gian có thể đổi trước khi frontend hiển thị. Hãy xác minh trạng thái cuối và dùng CloudWatch Logs hoặc các trường timestamp trong DynamoDB để xác nhận các giai đoạn đã hoàn tất.

## Ma trận kiểm thử

| ID | Bài kiểm thử | Kết quả mong đợi |
|---|---|---|
| T01 | Xác thực và phân quyền route | Đăng nhập hợp lệ; thiếu token nhận `401`; User gọi Admin nhận `403` |
| T02 | Luồng tài liệu an toàn | Cuối cùng `SAFE`; object chỉ ở bucket clean; tải xuống được |
| T03 | Luồng malware | Cuối cùng `REJECTED`; object ở bucket rejected; không tải xuống được |
| T04 | Luồng quyết định AI | Lưu trường rủi ro và hành động hệ thống tạo đúng nhánh |
| T05 | Admin kiểm duyệt thủ công | Một `ALLOW` và một `REJECT` hoàn tất qua Decision Lambda |
| T06 | Cô lập quyền sở hữu | Người dùng khác nhận `404`, không thể tải hoặc xóa |
| T07 | Luồng xóa | Object S3 bị xóa; metadata thành `DELETED`; danh sách ẩn item |
| T08 | Idempotency và đồng thời | Sự kiện lặp bị bỏ qua; quyết định Admin đồng thời có một conflict |
| T09 | Frontend và CloudFront | HTTPS, callback, deep link, CORS, upload và logout hoạt động |
| T10 | Quan sát và xử lý lỗi | Log liên kết được bằng ID, không lộ secret hoặc nội dung nguy hiểm |

## T01 — Kiểm tra xác thực và phân quyền

1. Mở CloudFront URL và đăng nhập bằng `workshop-user-a`.
2. Xác nhận Cognito quay về `/callback` và các trang tài liệu tải được.
3. Đăng xuất và xác nhận route được bảo vệ bắt đầu lại quá trình đăng nhập.
4. Gọi `GET /documents` không có header `Authorization`; mong đợi `401 Unauthorized`.
5. Đăng nhập bằng `workshop-user-a` và gọi `GET /admin/documents`; mong đợi `403 Forbidden`.
6. Đăng nhập bằng `workshop-admin`; xác nhận trang và API Admin hoạt động.

Chỉ đạt khi giao diện trình duyệt và backend cho cùng kết quả phân quyền. Chỉ ẩn menu chưa được tính là đạt.

## T02 — Kiểm tra nhánh tài liệu an toàn

1. Đăng nhập bằng `workshop-user-a` và tải lên tệp an toàn.
2. Xác nhận `POST /upload-url` trả `201` và S3 `PUT` thành công mà không có Cognito authorization header.
3. Gọi `GET /documents/{documentId}/scan-result` đến khi có trạng thái cuối hoặc hết thời gian chờ của workshop.
4. Mở item trong `secure-documents`.

Các trường DynamoDB mong đợi gồm:

```json
{
  "status": "SAFE",
  "malwareStatus": "NO_THREATS_FOUND",
  "aiStatus": "COMPLETED",
  "decisionStatus": "COMPLETED",
  "finalDecision": "ALLOW",
  "bucket": "securedocs-clean-<account-id>",
  "currentPrefix": "documents",
  "sourceCleanupStatus": "COMPLETED"
}
```

Xác nhận `s3Key` bắt đầu bằng `documents/<documentId>/`, object clean tồn tại và `quarantineKey` cũ không còn trong bucket quarantine. Tạo URL tải xuống, tải tệp và so sánh kích thước với bản gốc.

## T03 — Kiểm tra từ chối malware

Tải tệp EICAR bằng luồng frontend thông thường. Không tải mã độc thật.

Kết quả mong đợi:

```json
{
  "status": "REJECTED",
  "malwareStatus": "THREATS_FOUND",
  "finalDecision": "REJECT",
  "finalDecisionSource": "GUARDDUTY",
  "bucket": "securedocs-rejected-<account-id>",
  "currentPrefix": "rejected"
}
```

Xác minh object chỉ tồn tại dưới `rejected/<documentId>/`, bản quarantine đã dọn dẹp và cả màn hình chi tiết lẫn endpoint download-url đều từ chối tải xuống. Quản trị viên không được đổi tài liệu này thành `ALLOW`.

## T04 — Kiểm tra phân tích AI và quyết định hệ thống

Tải các tài liệu vô hại có mức rủi ro thấp, trung bình và cao được kiểm soát. Vì kết quả mô hình sinh có thể thay đổi, hãy ghi lại `riskScore`, `riskLevel`, `aiConfidence`, `aiRecommendedAction`, `systemRecommendedAction` và tóm tắt thực tế của từng lần chạy.

Xác nhận:

- AI chỉ chạy sau khi `malwareStatus = NO_THREATS_FOUND`.
- Item đạt `AI_ANALYSIS_COMPLETED` trước khi Decision Lambda nhận xử lý.
- `ALLOW` kết thúc ở `SAFE`, `REJECT` ở `REJECTED` và `REVIEW` ở `MANUAL_REVIEW`.
- Lỗi parse hoặc model trở thành `AI_ERROR`; tài liệu không được phát hành âm thầm.
- Frontend hiển thị kết quả phân tích dưới dạng text và không render nội dung trích xuất chưa tin cậy thành HTML.

## T05 — Kiểm tra kiểm duyệt thủ công

Tạo hai tài liệu sạch đạt `MANUAL_REVIEW`.

Với tài liệu thứ nhất:

1. Mở trang Admin review.
2. Gửi `ALLOW` cùng lý do.
3. Xác nhận API trả `202` với `ADMIN_DECISION_READY`.
4. Chờ Decision Lambda rồi xác minh trạng thái cuối `SAFE`, `finalDecisionSource = ADMIN_REVIEW` và object dưới `documents/` trong bucket clean.

Với tài liệu thứ hai, gửi `REJECT` rồi xác minh `REJECTED` và object dưới `rejected/` trong bucket rejected.

Với cả hai item, kiểm tra `reviewedBy`, `adminReason`, `reviewedAt` cùng timestamp quyết định cuối. Lambda Admin review không được có quyền ghi hoặc xóa S3 trực tiếp.

## T06 — Kiểm tra cô lập quyền sở hữu

Dùng một `documentId` thuộc `workshop-user-a`, sau đó đăng nhập bằng `workshop-user-b` và gọi:

```text
GET    /documents/{documentId}
GET    /documents/{documentId}/scan-result
GET    /documents/{documentId}/download-url
DELETE /documents/{documentId}
```

Mọi request phải trả `404 Not Found`; item DynamoDB và object S3 không thay đổi. Danh sách của User B không được chứa item của User A. Lặp lại với ID ngẫu nhiên và xác nhận response không tiết lộ ID có tồn tại hay không.

## T07 — Kiểm tra xóa

Xóa một tài liệu thử nghiệm ở trạng thái cuối thuộc người dùng đang đăng nhập.

Xác nhận:

- API trả `204 No Content`.
- DynamoDB có `status = DELETED`, `deletedAt` và `updatedAt`.
- Object S3 hiện tại bị xóa khỏi bucket và prefix đã xác minh.
- `GET /documents` không còn hiển thị item.
- Lần xóa thứ hai không tạo lại metadata hoặc làm lộ chi tiết nội bộ.

Thử xóa tài liệu khi tiến trình AI hoặc decision đang sở hữu item. Mong đợi `409 Conflict` và xác minh Lambda xử lý vẫn hoàn tất bình thường.

## T08 — Kiểm tra retry và quyết định đồng thời

Dùng tính năng test của Lambda console để phát lại một sự kiện DynamoDB Stream đã hoàn tất cho Decision Lambda. Thay giá trị nhạy cảm bằng dữ liệu thử nghiệm workshop và không sửa item thật đã hoàn tất.

Conditional claim phải trả `SKIPPED` hoặc conditional conflict mà không sao chép hay xóa thêm object. Xác nhận vẫn chỉ có một object đích chính thức.

Với item `MANUAL_REVIEW`, gần như đồng thời gửi `ALLOW` và `REJECT` từ hai phiên Admin. Chính xác một request phải trả `202`; request còn lại trả `409 Conflict`. Vị trí S3 cuối và quyết định DynamoDB phải khớp request thành công.

## T09 — Kiểm tra CloudFront và trình duyệt

Xác minh qua CloudFront domain:

- HTTP chuyển hướng sang HTTPS.
- Refresh trực tiếp `/documents/<documentId>` và `/admin/documents` vẫn trả ứng dụng React.
- Cognito callback và sign-out dùng đúng allowed URL.
- API preflight và request có xác thực hoạt động với CloudFront origin.
- Presigned upload thành công cùng các S3 header bắt buộc.
- Truy cập trực tiếp object frontend trong S3 vẫn bị từ chối.
- Bản triển khai mới cập nhật `index.html` và không thiếu asset có hash.
- Developer tools không có lỗi mixed content, CORS, CSP hoặc JavaScript chưa xử lý.

## T10 — Kiểm tra log và xử lý lỗi

Với từng tài liệu thử nghiệm, liên kết:

```text
API Gateway request ID
Lambda request ID
documentId
ownerId
GuardDuty finding hoặc kết quả quét
Timestamp trạng thái DynamoDB
S3 key nguồn và đích
```

Kiểm tra CloudWatch log group của upload, malware result, AI analysis, decision, document API và Admin. Log có thể chứa mã định danh cùng thay đổi trạng thái nhưng không được chứa access token, AWS credential, presigned URL, mật khẩu, toàn bộ nội dung tài liệu hoặc text trích xuất chưa tin cậy.

Nếu bài kiểm thử đạt `MALWARE_SCAN_ERROR`, `AI_ERROR` hoặc `DECISION_ERROR`, xác nhận object không thể tải xuống, object nguồn được giữ lại khi đích chưa xác minh và item có trường lỗi hữu ích nhưng đã làm sạch. Khôi phục quyền hoặc cấu hình lỗi trước khi thử lại trong môi trường workshop cô lập.

## Ghi nhận kết quả

Dùng mẫu sau để lưu bằng chứng:

| Test ID | Ngày/giờ | Tài khoản | Document ID | Mong đợi | Thực tế | Bằng chứng | Pass/Fail |
|---|---|---|---|---|---|---|---|
| T01 |  |  | N/A |  |  | Screenshot/log |  |
| T02 |  |  |  | `SAFE` |  | DynamoDB/S3/log |  |
| T03 |  |  |  | `REJECTED` |  | GuardDuty/S3/log |  |
| T04 |  |  |  | Nhánh AI |  | DynamoDB/log |  |
| T05 |  |  |  | Quyết định Admin |  | DynamoDB/S3/log |  |
| T06 |  |  |  | `404` |  | API response |  |
| T07 |  |  |  | `DELETED` |  | DynamoDB/S3 |  |
| T08 |  |  |  | Một quyết định |  | API/log |  |
| T09 |  |  | N/A | Luồng trình duyệt |  | Bằng chứng trình duyệt |  |
| T10 |  |  |  | Log/lỗi an toàn |  | CloudWatch |  |

Workshop đạt yêu cầu khi mọi bài bắt buộc thành công, nội dung không an toàn không bao giờ tải xuống được, ranh giới quyền sở hữu và Admin được giữ, mỗi tài liệu cuối chỉ có một vị trí chính thức, dọn dẹp nguồn hoàn tất hoặc được đánh dấu chờ xử lý và không có secret trong log hay mã frontend.
