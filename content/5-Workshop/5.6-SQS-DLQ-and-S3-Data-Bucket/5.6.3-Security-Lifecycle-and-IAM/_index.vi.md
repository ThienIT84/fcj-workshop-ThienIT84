---
title : "Security, lifecycle và IAM"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---


## Mục tiêu

Xác minh bảo mật S3 bucket, trạng thái lifecycle/cost-control và IAM permissions cho backend role và worker role.

## Bước 1: Xác minh bucket security

S3 Data Bucket chứa security evidence, vì vậy Block Public Access phải được bật và default encryption nên active.

![S3 security settings](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-06-s3-security-settings.png)

## Bước 2: Kiểm tra lifecycle và cost-control status

Lifecycle rule giúp expire temporary raw evidence hoặc chuyển object cũ sang storage class rẻ hơn. Nếu rule chưa cấu hình đầy đủ, giữ phần này là cost-control .

![S3 lifecycle](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-09-s3-lifecycle.png)

## Bước 3: Xác minh backend IAM permissions

Backend role `socai-dev-backend-role` cần được phép gửi message vào SQS main queue và ghi object vào S3 Data Bucket.

![Backend IAM policy for SQS and S3](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-10-iam-policy-backend-sqs-s3.png)

## Bước 4: Xác minh worker IAM permissions

Worker role `socai-dev-ai-worker-role` cần được phép receive/delete SQS message và đọc S3 object cần thiết.

![Worker IAM policy for SQS and S3](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-11-iam-policy-worker-sqs-s3.png)
