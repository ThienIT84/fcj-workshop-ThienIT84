---
title : "Alarm, SNS và Audit Status"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.10.3. </b> "
---

## Mục tiêu

Ghi lại CloudTrail, SNS và DLQ alarm evidence mà không claim quá phạm vi vận hành production.

CloudTrail trail đã có evidence cấu hình và log delivery, nhưng cần verify phạm vi production trước khi claim vận hành hoàn chỉnh.

![CloudTrail status](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-65-cloudtrail-placeholder.png)

![CloudTrail event history](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-66-cloudtrail-event-history.png)

![CloudTrail trail after config](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-67-cloudtrail-trail-after-config.png)

Các screenshot CloudTrail bổ sung của Tín có thể dùng như configured AWS evidence khi tồn tại.

![CloudTrail trail detail](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-22-cloudtrail-trail-detail.png)

![CloudTrail S3 log object](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-24-cloudtrail-s3-log-object.png)

SNS subscription đã có evidence `Confirmed`, nhưng không claim alarm email delivery nếu chưa có screenshot email trong mailbox.

![SNS status](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-68-sns-placeholder.png)

![SNS subscription after config](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-70-sns-subscription-after-config.png)

![SNS subscription after config](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-70-sns-subscription-after-config.png)

![SNS subscription confirmed](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-25-sns-subscription-confirmed.png)

DLQ CloudWatch Alarm đã có evidence state và actions enabled, nhưng không claim SNS action target hoặc email delivery nếu screenshot không hiển thị.

![CloudWatch DLQ alarm detail](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-tin-26-cloudwatch-dlq-alarm-detail.png)

SNS test notification screenshot chỉ nên dùng làm evidence cho đúng phạm vi test nhìn thấy trong ảnh.

![SNS test notification](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-71-sns-test-notification.png)
