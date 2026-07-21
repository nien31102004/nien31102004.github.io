---
title : "Prepare Document Extraction Libraries"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

The AI Analysis Lambda function needs to extract text content from `TXT`, `PDF`, and `DOCX` documents before sending the extracted data to Amazon Bedrock.

In this project:

+ `TXT` files are decoded directly using Python.
+ `PDF` files are processed using the `pypdf` library.
+ `DOCX` files are processed using the `python-docx` library.

Create a new directory on your local machine or in AWS CloudShell:

```bash
mkdir ai-analysis-package
cd ai-analysis-package
```

Install the required libraries into the current directory:

```bash
pip install pypdf python-docx -t .
```

![Install libraries](/images/5-Workshop/5.6/5.6.1/5.6.1.1.png)

Copy the `lambda_function.py` file containing the AI Analysis Lambda source code into the same directory.

The directory structure should look like the following:

```text
ai-analysis-package/
├── lambda_function.py
├── pypdf/
├── docx/
├── lxml/
└── Other dependency libraries
```

Compress all files and folders inside the directory into a ZIP package:

```bash
zip -r ai-analysis-package.zip .
```

![Create ZIP package](/images/5-Workshop/5.6/5.6.1/5.6.1.2.png)

{{% notice warning %}}
Compress only the files and folders inside the `ai-analysis-package` directory. Do **not** compress the parent directory; otherwise, AWS Lambda may not be able to locate the `lambda_function.py` file.
{{% /notice %}}

{{% notice note %}}
If the `pypdf` library is missing, the Lambda function cannot process PDF documents. If the `python-docx` library is missing, the Lambda function cannot extract content from DOCX documents.
{{% /notice %}}