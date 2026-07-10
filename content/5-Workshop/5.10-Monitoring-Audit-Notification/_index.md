---
title: "Monitoring, Audit and Notification"
date: 2024-01-01
weight: 10
chapter: false
pre: "<b>5.10. </b>"
---

## Objective

This section documents monitoring, audit, and notification evidence for the Hybrid SOC workshop. Phu provides the dashboard, CloudWatch, CloudTrail, and SNS monitoring narrative; Tin provides supplementary configured AWS evidence for SQS, CloudWatch SQS/DLQ metrics, CloudTrail trail, SNS subscription, and DLQ alarm where available.

```text
Frontend / CloudFront / WAF
Backend / Worker
SQS / DLQ / S3 / RDS
CloudWatch / CloudTrail / SNS
```

Evidence scope must be described precisely. CloudTrail trail evidence has been captured, including logging and log delivery, but the production audit scope should be verified before claiming full operational coverage. SNS subscription evidence shows a `Confirmed` email subscription, but alarm email delivery is not claimed without mailbox evidence. DLQ CloudWatch Alarm evidence shows the alarm state and enabled actions, but SNS action target and email delivery are not claimed unless visible in the captured evidence.
