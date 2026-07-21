---
title : "Chuẩn bị thư viện trích xuất tài liệu"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

Lambda phân tích AI cần trích xuất nội dung văn bản từ các tài liệu `TXT`, `PDF` và `DOCX` trước khi gửi dữ liệu đến Bedrock Mantle.

Trong dự án này:

+ File `TXT` được giải mã trực tiếp bằng Python.
+ File `PDF` được trích xuất bằng thư viện `pypdf`.
+ File `DOCX` được trích xuất bằng thư viện `python-docx`.

Tạo một thư mục mới trên máy tính hoặc trong AWS CloudShell:

```bash
mkdir ai-analysis-package
cd ai-analysis-package
```

Cài đặt các thư viện vào thư mục hiện tại:

```bash
pip install pypdf python-docx -t .
```

![Install libraries](/images/5-Workshop/5.6/5.6.1/5.6.1.1.png)

Sao chép file `lambda_function.py` chứa mã nguồn Lambda phân tích AI vào cùng thư mục.

Cấu trúc thư mục sau khi hoàn thành:

```text
ai-analysis-package/
├── lambda_function.py
├── pypdf/
├── docx/
├── lxml/
└── các thư viện phụ thuộc
```

Nén toàn bộ nội dung bên trong thư mục thành file ZIP:

```bash
zip -r ai-analysis-package.zip .
```

![Create ZIP package](/images/5-Workshop/5.6/5.6.1/5.6.1.2.png)

{{% notice warning %}}
Phải nén trực tiếp các file và thư mục bên trong `ai-analysis-package`. Không nén thư mục cha, nếu không Lambda có thể không tìm thấy file `lambda_function.py`.
{{% /notice %}}

{{% notice note %}}
Nếu thiếu `pypdf`, Lambda không thể xử lý PDF. Nếu thiếu `python-docx`, Lambda không thể trích xuất nội dung tài liệu DOCX.
{{% /notice %}}