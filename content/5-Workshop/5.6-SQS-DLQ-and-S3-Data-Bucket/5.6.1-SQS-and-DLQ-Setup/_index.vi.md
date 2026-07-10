---
title : "Cấu hình SQS và DLQ"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---


## Mục tiêu

Tạo và xác minh SQS main queue, queue configuration, DLQ và redrive policy dùng cho pipeline bất đồng bộ.

## Bước 1: Xác minh main queue

Mở Amazon SQS và chọn `socai-dev-normalized-zeek-events-queue`. Kiểm tra queue name, type, region, URL và ARN.

![SQS main queue detail](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-01-sqs-main-queue-detail.png)

## Bước 2: Kiểm tra queue configuration

Kiểm tra các giá trị ảnh hưởng đến runtime behavior:

- Visibility timeout quy định message bị ẩn bao lâu sau khi worker nhận để xử lý.
- Message retention quy định SQS giữ message chưa bị delete trong bao lâu.
- Encryption cần được bật cho queue.

![SQS main queue configuration](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-02-sqs-main-configuration.png)

## Bước 3: Xác minh DLQ

Mở DLQ `socai-dev-normalized-zeek-events-dlq`. Queue này lưu message xử lý lỗi sau số lần retry đã cấu hình.

![SQS DLQ detail](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-03-sqs-dlq-detail.png)

## Bước 4: Xác minh redrive policy

Quay lại main queue và xác nhận redrive policy trỏ đến DLQ. Max receive count cần được ghi lại để worker behavior dễ dự đoán.

![SQS redrive policy](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-04-sqs-redrive-policy.png)
