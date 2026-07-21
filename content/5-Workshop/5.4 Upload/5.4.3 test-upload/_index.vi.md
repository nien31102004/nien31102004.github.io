---
title : "Kiểm tra Presigned URL"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

Sử dụng Postman để kiểm tra API tạo Presigned URL trước khi tích hợp vào Frontend.

Mở Postman và tạo request mới với phương thức:

```text
POST
```

Nhập URL của API:

```text
https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/upload-url
```


Trong tab **Authorization**, chọn **Bearer Token** và nhập JWT Token nhận được sau khi đăng nhập bằng Amazon Cognito.


Trong tab **Headers**, thêm:

| Key | Value |
| --- | --- |
| `Content-Type` | `application/json` |

Trong tab **Body**, chọn **raw**, chọn định dạng **JSON** và nhập:

```json
{
 "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
"fileName": "3-Phieu-Theo-doi-Tien-do-TTTN (1).docx",
"fileSize":48501
}
```

![Create request](/images/5-Workshop/5.4/5.4.3/5.4.3.1.png)

Chọn **Send** để gửi yêu cầu.

Khi API hoạt động thành công, hệ thống trả về:

```json
{
    "message": "Đã tạo URL upload",
    "documentId": "55769149-7ab8-4f14-8514-c163b83e37c9",
    "uploadUrl": "https://secure-ai-document-storage.s3.amazonaws.com/quarantine/c9da755c-e081-70bf-8573-46d5e00cd2e7/55769149-7ab8-4f14-8514-c163b83e37c9/3-Phieu-Theo-doi-Tien-do-TTTN__1_.docx?AWSAccessKeyId=ASIA52W5LQ55XZWXGZ6K&Signature=plYYryGt9IntYSPu8ZKmSdJR5DM%3D&content-type=application%2Fvnd.openxmlformats-officedocument.wordprocessingml.document&x-amz-meta-document-id=55769149-7ab8-4f14-8514-c163b83e37c9&x-amz-meta-user-id=c9da755c-e081-70bf-8573-46d5e00cd2e7&x-amz-security-token=IQoJb3JpZ2luX2VjENv%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLXNvdXRoZWFzdC0xIkcwRQIgekJQLOcEtcm82gktkIVSTLt4dCeYSB%2FpzGTkFeu6z%2FgCIQCI95W5HK%2F8R9Wy4E5bmhJ4rZw544GvgGlawjf%2Bw3sg4iqmBAik%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDk1MDcyNTc0MDQxMSIMlScPYXR7Vk9Qd%2FuaKvoDPGTT1oAA0YCHJU1XCyD9b8upMiVqO4IMMyap9GqyyfA7IQ%2BF3v%2F3n2R9PNAOpw9s%2FxCE1fzgcB4U0sUj29MJy5gc55YF%2Fi0sThBjL8Vz05Tjkz%2Bnvwz7ohYwvoW%2BdjlLjM6oO7jisiZbAA1NLpbOl2GpNAngQKacL9%2FIu%2FJVbRe9VuPatw4StOrlIicYVitNqcFnc%2B5eeoQnOtWPSpsK9dOS2MeqFCHqBWPZYL%2B0kX5g%2BE9oE5%2FLMOCxJl7CQIxUn8%2Bz3QTvZoFTTikoFH7EkfPEBwclFQHqBSaHFJCpVt6mhMAoy3oPWG9nVGSTu%2FNR3x6elBJ9tv4xW8VlJa6e2XRvBeei4yAFcOF9ZeJfuWeCfOmfRSAGa%2ByEphGSWiHRu3b%2BOxA6lqjSpICVVDktJhPH1kiJnWBFuMMshbu9baCjodrY5SHpbub%2FCTvbdXjHR4b1XEgxnvbRoDHBQoXuDv2SNe4mhIjfGAqcT9dn%2F03QDmCXE3gy3H0KcArUU3iW5sweLMJb8gt9gwPZJa71CbAOrj%2FHOfDnK3aLdESVyO5hvl4BxW8e35IiOP%2F1BpZLXph0IyE8gpgxbJsjHwe4K7Vt%2FkVMQidJtUjLQHcjgxkyS8%2BuJPgxzuItEsVN9Yvh84AcaAJVoRkG416E96bxi%2FhVPPhiYW8ueBAwh%2FP30gY6ogHCu5UIX3CztoElF%2BTsSPtLp%2BRA22mNixyEdCFZqGpgIymaLmn88fLZQmtZALzKxNXq%2FOX788JN2ZiMJRJnDcKlSV4emftaSJnTw2xJbFrfNANDtb81xfBzJwKKiEKc48rx86vJYVd2fGMIiIV5swKLEmFhQsI2kqnZajQnhg96%2BVQu8aCKCTtvwbT9TjUEmIlQ2dTrwWjwu6jqt6dd5Lugni4%3D&Expires=1784543924",
    "method": "PUT",
    "expiresIn": 300,
    "requiredHeaders": {
        "Content-Type": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
        "x-amz-meta-document-id": "55769149-7ab8-4f14-8514-c163b83e37c9",
        "x-amz-meta-user-id": "c9da755c-e081-70bf-8573-46d5e00cd2e7"
    }
}
```

![Presigned URL response](/images/5-Workshop/5.4/5.4.3/5.4.3.2.png)

Mở Amazon DynamoDB và kiểm tra bảng `SecureDocuments`. Một bản ghi mới sẽ được tạo với trạng thái `UPLOADED`.

![DynamoDB record](/images/5-Workshop/5.4/5.4.3/5.4.3.3.png)

{{% notice note %}}
Presigned URL chỉ có hiệu lực trong khoảng thời gian được cấu hình. Với giá trị `900`, URL hết hạn sau 15 phút và cần được tạo lại nếu chưa tải file lên.
{{% /notice %}}