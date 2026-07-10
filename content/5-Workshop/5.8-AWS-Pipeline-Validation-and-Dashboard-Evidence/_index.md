---
title : "AWS Pipeline Validation and Dashboard Evidence"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

This section validates the AWS pipeline end-to-end without depending on a Local Lab. Instead of building a separate attack and sensor environment, the workshop uses a test payload or replay event to validate the cloud flow:

```text
Backend ingestion API
-> S3 raw event / SQS message
-> AI Worker
-> RDS final alert
-> Backend API or Dashboard
```

Local Lab is removed from the scope of this section so the workshop can focus on deploying and validating AWS services. Evidence is generated with a sample/replay event that contains an HTTP SQL Injection payload.

#### AWS-only validation flow

```text
Test HTTP security event
-> Backend API /api/events/http/async
-> Store raw event evidence in S3
-> Send normalized event to SQS
-> AI Worker consumes message
-> AI2B detects SQLI
-> Fusion creates Critical alert
-> Store final alert in RDS
-> /api/alerts/latest returns latest alert
```

The goal of this section is to prove an AWS thin-slice end-to-end path: the event enters the backend, evidence appears in S3/SQS, the worker turns it into an alert, RDS stores the result, and the API/Dashboard can read the latest alert.

#### Create a test event and verify SQS/S3 evidence

This screenshot proves that the backend or replay test has pushed the event into the AWS data pipeline. Evidence can be an SQS message, S3 object key, message ID, or a `queued` response.

```bash
curl -i -X POST "http://127.0.0.1:8000/api/events/http/async" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "workshop-test",
    "log_type": "http",
    "timestamp": "2026-07-06T08:52:13Z",
    "src_ip": "0.0.0.0",
    "dst_ip": "0.0.0.0",
    "method": "GET",
    "uri": "/search?q=%27%20OR%201%3D1--",
    "user_agent": "workshop-e2e-test"
  }'
```

The expected response can include:

```json
{
  "status": "queued",
  "event_id": "evt-...",
  "message_id": "...",
  "s3_raw_key": "..."
}
```

![SQS and S3 evidence for AWS pipeline](/images/5-Workshop/5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/5.8.1.png)

#### Verify Worker/RDS evidence

This screenshot proves that the AI Worker processed an event from SQS. The important log line is `processed message event_id=... alert_id=...`. If the screenshot also includes an RDS query, that is useful; otherwise, section 5.7 already contains separate RDS row evidence.

```bash
echo "Screenshot: 5.8.2"
sudo journalctl -u socai-ai-worker --no-pager | grep "processed message" | tail -n 5
```

Optional query:

```sql
SELECT id, attack_type, severity, risk_score, technique_id
FROM final_alerts
ORDER BY created_at DESC
LIMIT 5;
```

If the actual table is `alerts`, replace `final_alerts` with `alerts`.

![Worker processed event and RDS validation](/images/5-Workshop/5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/5.8.2.png)

#### Verify the API/Dashboard alert

This screenshot proves that the final alert appears in the API/Dashboard layer. In the current screenshot, the latest alert should contain these key values:

| Field | Expected value |
|---|---|
| `id` | `evt-...` |
| `attack_type` | `SQL Injection` |
| `severity` | `Critical` |
| `risk_score` | `97` |
| MITRE | `T1190 Exploit Public-Facing Application` |
| AI2B classification | `SQLI` |
| Fusion mode | `DEGRADED_AI2B_ONLY` |

```bash
echo "Screenshot: 5.8.3"
curl -s http://127.0.0.1:8000/api/alerts/latest | python3 -m json.tool
```

![Latest alert with AI analysis and MITRE mapping](/images/5-Workshop/5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/5.8.3.png)

#### Troubleshooting

| Issue | Symptom | Resolution |
|---|---|---|
| Backend does not queue the event | Response does not include `queued`, `message_id`, or `s3_raw_key` | Check the backend service, IAM permissions for SQS/S3, and queue/bucket environment variables |
| S3 object is missing | Response includes `s3_raw_key` but the bucket has no object | Check bucket name, region, IAM `s3:PutObject`, and the prefix being used |
| SQS message is missing | Backend receives the request but the worker has no new log | Check queue URL, region, and IAM `sqs:SendMessage`/`sqs:ReceiveMessage` |
| Worker does not process the event | No `processed message` log | Check the `socai-ai-worker` service, queue URL, visibility timeout, and dead-letter queue |
| Worker cannot read Secrets Manager | Log includes `AccessDeniedException GetSecretValue` | Attach `secretsmanager:GetSecretValue` permission to the worker role |
| Worker has an RDS error | Log includes a timeout or insert failure | Check the RDS Security Group, Secrets Manager, table name, and transaction commit |
| API does not return latest alert | `/api/alerts/latest` returns an error or empty response | Check the backend DB connection, latest alert query, and RDS data |
| Dashboard does not show the alert | API has data but UI does not display it | Check API base URL, filters, cache, or mock/realtime mode |

#### Conclusion

The validation result shows that the AWS pipeline works as a thin-slice end-to-end path: the test event goes through backend ingestion, is pushed to SQS/S3, is processed by the worker into a final alert, is stored in RDS, and can be read back by the API/Dashboard.

This section does not claim Local Lab or production HA. Auto Scaling, CloudTrail, and SNS email notification are separate extension sections handled by other team members.

> Do not include DB passwords, secret values, access keys, full account IDs, full ARNs, private endpoints, real internal IPs, `.pem` files, tokens, or un `.env` files in screenshots or workshop content.
