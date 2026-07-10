---
title: "Monitoring, Audit and Notification"
date: 2024-01-01
weight: 10
chapter: false
pre: "<b>5.10. </b>"
---

## Mục tiêu

Phần này ghi lại monitoring, audit và notification evidence cho workshop Hybrid SOC. Phú cung cấp monitoring/audit/notification plan tổng thể cho dashboard, CloudWatch, CloudTrail và SNS; Tín cung cấp supplementary configured AWS evidence cho SQS, CloudWatch SQS/DLQ metrics, CloudTrail trail, SNS subscription và DLQ alarm khi có.

```text
Frontend / CloudFront / WAF
Backend / Worker
SQS / DLQ / S3 / RDS
CloudWatch / CloudTrail / SNS
```

Phạm vi evidence phải được mô tả chính xác. CloudTrail trail đã có evidence cấu hình và log delivery, nhưng cần verify phạm vi production trước khi claim vận hành hoàn chỉnh. SNS subscription đã có evidence `Confirmed`, nhưng không claim alarm email delivery nếu chưa có screenshot email trong mailbox. DLQ CloudWatch Alarm đã có evidence state và actions enabled, nhưng không claim SNS action target hoặc email delivery nếu screenshot không hiển thị.
