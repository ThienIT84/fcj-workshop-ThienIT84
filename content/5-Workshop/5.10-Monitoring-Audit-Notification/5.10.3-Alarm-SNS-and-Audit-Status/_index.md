---
title : "Alarm, SNS and Audit Status"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.3. </b> "
---

## Objective

Document CloudTrail, SNS, and DLQ alarm evidence without over-claiming production operation.

CloudTrail trail evidence has been captured, including logging and log delivery, but the production audit scope should be verified before claiming full operational coverage.

![CloudTrail status](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-65-cloudtrail-placeholder.png)

![CloudTrail event history](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-66-cloudtrail-event-history.png)

![CloudTrail trail after config](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-67-cloudtrail-trail-after-config.png)

Tin's supplementary CloudTrail screenshots can be used as configured AWS evidence when present.

![CloudTrail trail detail](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-22-cloudtrail-trail-detail.png)

![CloudTrail S3 log object](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-24-cloudtrail-s3-log-object.png)

SNS subscription evidence shows a `Confirmed` email subscription, but alarm email delivery is not claimed without mailbox evidence.

![SNS status](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-68-sns-placeholder.png)

![SNS subscription after config](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-70-sns-subscription-after-config.png)

![SNS subscription after config](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-70-sns-subscription-after-config.png)

![SNS subscription confirmed](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-25-sns-subscription-confirmed.png)

DLQ CloudWatch Alarm evidence shows the alarm state and enabled actions, but SNS action target and email delivery are not claimed unless visible in the captured evidence.

![CloudWatch DLQ alarm detail](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-26-cloudwatch-dlq-alarm-detail.png)

The SNS test notification screenshot can be included only as evidence for the visible test scope.

![SNS test notification](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-71-sns-test-notification.png)
