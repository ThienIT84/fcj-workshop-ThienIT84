---
title : "Troubleshooting and Evidence Status"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.10.4. </b> "
---

## Browser and frontend evidence

DevTools screenshots help separate frontend delivery issues from backend or pipeline issues.

![DevTools static assets 200](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-57-devtools-static-assets-200.png)

![DevTools API call mock or real](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-58-devtools-api-call-mock-or-real.png)

![DevTools console warnings](/images/5-Workshop/5.10-Monitoring-Audit-Notification/w-phu-60-devtools-console-warnings.png)

## Troubleshooting checklist

| Symptom | Check |
|---|---|
| Dashboard loads but data is stale | Confirm mock/replay mode, API endpoint, and CloudFront cache |
| Static files fail | Check CloudFront, S3 root objects, and WAF |
| API calls fail | Check backend health, CORS, and security group rules |
| Queue backlog grows | Check worker logs and SQS metrics |
| DLQ has messages | Inspect payload, worker error, and alarm state |
| Alarm action is unclear | Do not claim notification delivery without visible SNS or mailbox evidence |
| Audit scope is unclear | Verify CloudTrail trail region, management/data events, and log delivery |

