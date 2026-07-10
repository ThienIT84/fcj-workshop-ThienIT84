---
title : "Backend Evidence và Troubleshooting"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.6.5. </b> "
---


## Mục tiêu

Ghi lại backend integration evidence đã có cho đường đi SQS, đồng thời giữ rõ phần S3 object evidence còn thiếu. Cách viết này tránh claim vượt quá những gì screenshot chứng minh.

## Plan sử dụng evidence

| Screenshot | Cách dùng |
|---|---|
| `w-tin-14-async-api-response-queued.png` | Dùng như worker/backend follow-up evidence. Log nhìn thấy worker service đang active, nhưng cũng có lỗi Secrets Manager `AccessDenied`, nên ảnh này thuộc cả phần troubleshooting. |
| `w-tin-15-sqs-message-after-backend.png` | Dùng như SQS evidence. Ảnh cho thấy received message body với normalized HTTP event sau backend ingestion. |
| `w-tin-16-s3-object-after-backend.png` | Hiện chưa có. Không reference ảnh này và không claim backend `put_object` evidence cho đến khi screenshot tồn tại. |
| `w-tin-17-backend-sqs-s3-log.png` | Dùng như backend API evidence. Ảnh cho thấy `POST /api/events/http/async` trả `202 Accepted` cho evidence event. |

## Backend API đã accepted event

Log này chứng minh backend đã nhận async HTTP event request và trả `202 Accepted`. Đây là evidence cho ingestion endpoint, nhưng chưa tự chứng minh mọi downstream step đã hoàn tất.

![Backend async API accepted event](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-17-backend-sqs-s3-log.png)

## SQS message được tạo sau backend ingestion

Screenshot này hiển thị message body nhận từ SQS. Event có schema version, event type, event ID, timestamp, source/destination IP và HTTP evidence. Ảnh này hỗ trợ claim rằng backend ingestion có thể tạo queue message.

![SQS message after backend ingestion](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-15-sqs-message-after-backend.png)

## Worker follow-up và blocker hiện tại

Worker service đang enabled và active, nhưng log có `AccessDeniedException` với `secretsmanager:GetSecretValue`. Đây là troubleshooting evidence quan trọng: đường SQS đã có evidence, nhưng worker-to-RDS processing vẫn cần quyền Secrets Manager đúng trước khi claim end-to-end pipeline hoàn chỉnh.

![Worker log and Secrets Manager access issue](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-14-async-api-response-queued.png)

## Evidence còn thiếu

Screenshot S3 object evidence vẫn chưa có. Cho đến khi `w-tin-16-s3-object-after-backend.png` tồn tại, phần backend `put_object` validation vẫn để pending.

## Validation checklist

| Check | Trạng thái |
|---|---|
| Main queue exists | Đã cover ở 5.6.1 |
| DLQ exists | Đã cover ở 5.6.1 |
| S3 Data Bucket exists | Đã cover ở 5.6.2 |
| IAM policies are prepared | Đã cover ở 5.6.3 |
| Manual SQS/S3 validation works | Đã cover ở 5.6.4 |
| Backend async endpoint accepts event | Có evidence trong `w-tin-17` |
| Backend-created SQS message exists | Có evidence trong `w-tin-15` |
| Backend-created S3 object exists | Pending; screenshot `w-tin-16` chưa có |
| Worker can read DB secret | Đang bị chặn bởi `secretsmanager:GetSecretValue` AccessDenied trong `w-tin-14` |

## Troubleshooting

| Symptom | Check |
|---|---|
| Backend bị AccessDenied với SQS | Kiểm tra `socai-dev-backend-role` có `sqs:SendMessage` |
| Backend bị AccessDenied với S3 | Kiểm tra `socai-dev-backend-role` có `s3:PutObject` cho data bucket |
| Queue không nhận message | Kiểm tra queue URL, region và backend logs |
| Worker không đọc được DB secret | Thêm quyền `secretsmanager:GetSecretValue` cho worker role và kiểm tra secret ARN |
| Worker retry message | Kiểm tra visibility timeout, worker logs và `DeleteMessage` |
| Message vào DLQ | Inspect failed payload và worker errors trước khi replay |

