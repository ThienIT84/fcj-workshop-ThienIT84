---
title : "Manual SQS/S3 validation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---


## Mục tiêu

Validate SQS và S3 thủ công trước khi có backend integration evidence.

## Bước 1: Gửi manual SQS message

Dùng SQS console hoặc CloudShell để gửi test message vào `socai-dev-normalized-zeek-events-queue`. Bước này chứng minh queue nhận được message, nhưng không chứng minh backend integration.

![Manual SQS test](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-12-sqs-manual-test.png)

## Bước 2: Upload manual S3 object

Upload test object vào đúng Zeek HTTP prefix trong `socai-dev-data-************`. Bước này chứng minh bucket và prefix dùng được, nhưng không chứng minh backend đã gọi `put_object`.

![Manual S3 object test](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-13-s3-manual-object-test.png)

## Kết quả

Manual validation xác nhận SQS và S3 resources đã sẵn sàng cho backend integration. Evidence do backend thật tạo ra vẫn đang chờ.
