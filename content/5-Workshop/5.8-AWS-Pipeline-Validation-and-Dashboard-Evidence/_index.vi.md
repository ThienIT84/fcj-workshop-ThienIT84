---
title : "AWS Pipeline Validation and Dashboard Evidence"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

Section này xác thực pipeline AWS end-to-end mà không phụ thuộc vào Local Lab. Thay vì dựng môi trường tấn công và sensor riêng, workshop dùng test payload hoặc replay event để kiểm tra luồng cloud:

```text
Backend ingestion API
-> S3 raw event / SQS message
-> AI Worker
-> RDS final alert
-> Backend API hoặc Dashboard
```

Local Lab đã được bỏ khỏi phạm vi section này để workshop tập trung vào triển khai và kiểm tra các service AWS. Evidence được tạo bằng sample/replay event có payload HTTP SQL Injection để xác thực pipeline.

#### Luồng AWS-only validation

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

Mục tiêu của section này là chứng minh thin-slice end-to-end trên AWS: event được đưa vào backend, có evidence ở S3/SQS, worker xử lý thành alert, RDS lưu kết quả và API/Dashboard đọc lại được alert mới nhất.

#### Tạo test event và kiểm tra SQS/S3 evidence

Ảnh này chứng minh backend hoặc replay test đã đưa event vào AWS data pipeline. Evidence có thể là SQS message, S3 object key, message ID hoặc response `queued`.

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

Expected response có thể chứa:

```json
{
  "status": "queued",
  "event_id": "evt-...",
  "message_id": "...",
  "s3_raw_key": "..."
}
```

![SQS and S3 evidence for AWS pipeline](/images/5-Workshop/5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/5.8.1.png)

#### Kiểm tra Worker/RDS evidence

Ảnh này chứng minh AI Worker đã xử lý event từ SQS. Dòng log quan trọng là `processed message event_id=... alert_id=...`. Nếu screenshot có thêm RDS query thì càng tốt; nếu không, section 5.7 đã có ảnh RDS row riêng.

```bash
echo "Screenshot: 5.8.2"
sudo journalctl -u socai-ai-worker --no-pager | grep "processed message" | tail -n 5
```

Query optional:

```sql
SELECT id, attack_type, severity, risk_score, technique_id
FROM final_alerts
ORDER BY created_at DESC
LIMIT 5;
```

Nếu table thực tế là `alerts`, thay `final_alerts` bằng `alerts`.

![Worker processed event and RDS validation](/images/5-Workshop/5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/5.8.2.png)

#### Kiểm tra API/Dashboard alert

Ảnh này chứng minh final alert đã xuất hiện ở lớp API/Dashboard. Trong screenshot hiện tại, alert mới nhất có các thông tin chính:

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

| Lỗi | Dấu hiệu | Cách xử lý |
|---|---|---|
| Backend không queue event | Response không có `queued`, `message_id` hoặc `s3_raw_key` | Kiểm tra backend service, IAM permission tới SQS/S3 và biến môi trường queue/bucket |
| Không thấy S3 object | Response có `s3_raw_key` nhưng bucket không có object | Kiểm tra bucket name, region, IAM `s3:PutObject` và prefix đang dùng |
| Không thấy SQS message | Backend nhận request nhưng worker không có log mới | Kiểm tra queue URL, region và IAM `sqs:SendMessage`/`sqs:ReceiveMessage` |
| Worker không xử lý event | Không có log `processed message` | Kiểm tra service `socai-ai-worker`, queue URL, visibility timeout và dead-letter queue |
| Worker lỗi Secrets Manager | Log có `AccessDeniedException GetSecretValue` | Gắn permission `secretsmanager:GetSecretValue` cho worker role |
| Worker lỗi RDS | Log có timeout hoặc insert fail | Kiểm tra RDS Security Group, Secrets Manager, table name và transaction commit |
| API không trả latest alert | `/api/alerts/latest` lỗi hoặc empty | Kiểm tra backend DB connection, query latest alert và dữ liệu trong RDS |
| Dashboard không có alert | API có data nhưng UI không hiển thị | Kiểm tra API base URL, filter, cache hoặc mode mock/realtime |

#### Kết luận

Kết quả validation cho thấy pipeline AWS đã hoạt động theo thin-slice end-to-end: test event đi qua backend ingestion, được đẩy vào SQS/S3, worker xử lý thành final alert, RDS lưu kết quả và API/Dashboard đọc lại được alert mới nhất.

Phần này không claim Local Lab hoặc production HA. Auto Scaling, CloudTrail và SNS email notification là các phần mở rộng/section riêng do thành viên khác hoàn thiện.

> Không đưa DB password, secret value, access key, account ID, ARN đầy đủ, endpoint private, IP nội bộ thật, `.pem`, token hoặc file `.env` chưa  vào screenshot/workshop.
