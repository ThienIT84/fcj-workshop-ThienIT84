---
title : "CloudWatch SQS và DLQ Metrics"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.10.2. </b> "
---

## Mục tiêu

Dùng CloudWatch để quan sát SQS và DLQ metrics, đồng thời dùng CloudWatch Logs nếu backend hoặc worker log group đã được cấu hình.

CloudWatch metrics evidence giúp nhận biết queue depth, message age và DLQ activity.

![CloudWatch SQS DLQ metrics](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-19-cloudwatch-sqs-dlq-metrics.png)

CloudWatch log groups cho thấy centralized logs có sẵn để troubleshooting backend hoặc worker hay không.

![CloudWatch log groups](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-61-cloudwatch-log-groups.png)

Backend và worker log stream screenshot chỉ nên được mô tả là evidence cho đúng service đã chụp.

![CloudWatch backend log stream](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-62-cloudwatch-backend-log-stream.png)

![CloudWatch worker log stream](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-63-cloudwatch-worker-log-stream.png)

Nếu log groups chưa cấu hình đầy đủ, giữ wording  thay vì claim centralized logging hoàn chỉnh.

![CloudWatch ](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-64-cloudwatch.png.png)

