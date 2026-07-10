---
title : "Troubleshooting và Evidence Status"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.10.4. </b> "
---

## Browser và frontend evidence

DevTools screenshots giúp tách lỗi frontend delivery khỏi lỗi backend hoặc pipeline.

![DevTools static assets 200](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-57-devtools-static-assets-200.png)

![DevTools API call mock or real](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-58-devtools-api-call-mock-or-real.png)

![DevTools console warnings](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-60-devtools-console-warnings.png)

## Troubleshooting checklist

| Dấu hiệu | Cần kiểm tra |
|---|---|
| Dashboard load nhưng data cũ | Xác nhận mock/replay mode, API endpoint và CloudFront cache |
| Static files lỗi | Kiểm tra CloudFront, S3 root objects và WAF |
| API calls lỗi | Kiểm tra backend health, CORS và security group rules |
| Queue backlog tăng | Kiểm tra worker logs và SQS metrics |
| DLQ có messages | Inspect payload, worker error và alarm state |
| Alarm action chưa rõ | Không claim notification delivery nếu không thấy SNS hoặc mailbox evidence |
| Audit scope chưa rõ | Verify CloudTrail trail region, management/data events và log delivery |

