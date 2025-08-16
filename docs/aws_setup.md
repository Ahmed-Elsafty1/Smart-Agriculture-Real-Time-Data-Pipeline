# AWS Setup Guide

This document describes the step-by-step AWS setup for the **Smart Agriculture Real-Time Data Pipeline** project.

---

## 1. AWS IoT Core
- Created an IoT Thing (`farm_data`).
- Generated X.509 device certificates.
- Attached IoT Policy with permissions:
  - `iot:Connect`
  - `iot:Publish`
  - `iot:Subscribe`
  - `iot:Receive`

### IoT Policy Example
```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["iot:Connect"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["iot:Publish","iot:Subscribe","iot:Receive"], "Resource": "*" }
  ]
}
```

---

## 2. Kinesis Data Stream
- Created Kinesis stream `farm-stream`.
- Configured IoT Rule to forward messages from topic `farm/data` into the stream.

---

## 3. S3 Buckets
- Created bucket: `farm-processed-data`
- Structure:
  - `valid/` → Clean & validated records.
  - `valid/warnings/` → Records with warnings.
  - `invalid/` → Invalid or corrupted records.
- Enabled default encryption (AES256).

---

## 4. AWS Lambda
- Created Lambda function: `farm-data-processor`
- Runtime: Python 3.9
- Trigger: Kinesis Data Stream (`farm-stream`)

### Permissions (Lambda Execution Role)
- `s3:PutObject` (write processed records to S3)
- `sns:Publish` (send alerts)
- `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` (CloudWatch logging)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:PutObject"], "Resource": "arn:aws:s3:::farm-processed-data/*" },
    { "Effect": "Allow", "Action": ["sns:Publish"], "Resource": "arn:aws:sns:us-east-1:123456789012:farm_alerts" },
    { "Effect": "Allow", "Action": ["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents"], "Resource": "*" }
  ]
}
```

---

## 5. Amazon SNS
- Created topic: `farm_alerts`
- Subscribed email endpoint for notifications.
- Attached policy allowing Lambda to publish alerts.

---

## 6. AWS Glue
- Created **Glue Crawler** on `s3://farm-processed-data/`
- Created **Glue ETL Job** to transform/load into Redshift

### Glue Policy Example
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject",
    "redshift:*",
    "logs:*"
  ],
  "Resource": "*"
}
```

---

## 7. Amazon Redshift
- Created Redshift cluster and database.
- Designed **Star Schema** with:
  - `fact_sensor_readings`
  - `dim_location`
  - `dim_time`
  - `dim_weather`
  - `dim_soil`

---

## 8. Monitoring
- **CloudWatch**: Logs and metrics for Lambda, Kinesis, SNS.
- **OpenSearch**: Centralized log indexing for troubleshooting.

---

## 9. Security Best Practices
- Store AWS IoT certificates locally, never commit them to GitHub.
- Use Secrets Manager/SSM for sensitive values.
- Enable encryption (KMS/AES256) for S3, Kinesis, Redshift.
- Use least-privilege IAM policies.
