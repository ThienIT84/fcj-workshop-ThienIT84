---
title : "CloudWatch SQS and DLQ Metrics"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.10.2. </b> "
---

## Objective

Use CloudWatch to observe SQS and DLQ metrics, and use CloudWatch Logs where backend or worker log groups are configured.

The CloudWatch metrics evidence helps identify queue depth, message age, and DLQ activity.

![CloudWatch SQS DLQ metrics](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-19-cloudwatch-sqs-dlq-metrics.png)

CloudWatch log groups show whether centralized logs are available for backend or worker troubleshooting.

![CloudWatch log groups](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-61-cloudwatch-log-groups.png)

Backend and worker log stream screenshots should be described as available evidence only for the captured services.

![CloudWatch backend log stream](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-62-cloudwatch-backend-log-stream.png)

![CloudWatch worker log stream](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-63-cloudwatch-worker-log-stream.png)

If log groups are not fully configured, keep the  wording instead of claiming centralized logging is complete.

![CloudWatch ](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-64-cloudwatch.png.png)

