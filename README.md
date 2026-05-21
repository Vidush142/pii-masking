# Real-Time Data Masking Engine via AWS S3 Object Lambda

A production-ready AWS Lambda function designed for real-time, on-the-fly PII (Personally Identifiable Information) masking. Utilizing a zero-memory footprint streaming architecture, this application intercepts S3 object delivery pipelines to redact sensitive fields before they exit the secure cloud perimeter.

## 📐 Architectural Design
Instead of executing destructive batch ETL pipelines that alter data permanently on disk, this solution implements dynamic masking at the consumption layer using **AWS S3 Object Lambdas**.

- **Zero-Memory Footprint:** Uses Python generator expressions (`yield`) and stream buffers to handle files of arbitrary scales (GBs/TBs) safely within the strict memory constraints of serverless runtimes.
- **High-Throughput Regex Engine:** Pre-compiles byte-compiled regular expressions (`re.compile`) globally outside the request handler, minimizing compute overhead across continuous line iterations.

## 🔒 Masking Matrix
| Data Type | Detection Strategy | Output Format |
| :--- | :--- | :--- |
| **Email Address** | Standard RFC 5322 Byte Regular Expression | `[REDACTED_EMAIL]` |
| **US SSN** | Standard 3-2-4 digit delimiters | `XXX-XX-XXXX` |
| **Credit Cards** | 16-Digit standard grouping structures | `XXXX-XXXX-XXXX-1234` (Retains final 4 digits) |

## 🛠️ Tech Stack & Prerequisites
- **Runtime Environment:** Python 3.9+ (Fully compatible up to 3.13)
- **AWS Services:** Amazon S3, S3 Access Points, S3 Object Lambda, AWS CloudWatch
- **Core Engineering Libraries:** `boto3`, `requests` (stream-enabled)

## 🚀 Deployment Instructions

### 1. Package the Lambda
Because this script utilizes the external `requests` library, it must be packaged into a zip file with its dependencies before deployment to AWS:

```bash
# Install dependencies to a target deployment directory
pip install --target ./package requests

# Add the source file to the deployment root
cp src/lambda_function.py ./package/

# Build deployment package
cd package
zip -r ../pii_masking_lambda.zip .
