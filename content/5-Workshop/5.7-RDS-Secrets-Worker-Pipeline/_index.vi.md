---
title : "RDS Secrets Worker Pipeline"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

Trong section này, chúng ta kiểm tra phần lưu trữ và xử lý phía backend của pipeline AWS. RDS PostgreSQL dùng để lưu final alerts sau khi worker xử lý message từ SQS. Secrets Manager dùng để quản lý thông tin kết nối database thay vì hard-code credential trong source code.

RDS trong workshop này chạy Single-AZ để giảm chi phí demo. Multi-AZ là hướng mở rộng production-style và chưa được triển khai trong môi trường hiện tại.

#### Luồng xử lý

```text
SQS Main Queue
-> AI Worker receive_message
-> Parse normalized security event
-> AI/Fusion processing
-> Insert final alert into RDS PostgreSQL
-> Delete SQS message after success
-> Backend API /api/alerts/latest reads latest alert
-> Dashboard/API evidence
```

Worker chỉ xóa message khỏi SQS sau khi xử lý thành công và ghi được final alert. Cách này giúp giảm rủi ro mất event khi worker lỗi giữa chừng.

#### RDS Single-AZ và private access

RDS PostgreSQL được cấu hình ở chế độ private access. Database không mở public access và chỉ cho phép traffic PostgreSQL 5432 từ Security Group của backend/worker. Đây là baseline bảo mật quan trọng để database không bị expose trực tiếp ra Internet.

RDS PostgreSQL đang available và chạy Single-AZ để giảm chi phí demo.

![RDS PostgreSQL Single-AZ summary](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.1.png)

RDS sử dụng private connectivity, public access disabled, endpoint và port 5432.

![RDS private connectivity and endpoint](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.2.png)

Security Group chỉ cho backend/worker truy cập PostgreSQL 5432.

![RDS security group inbound rule](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.3.png)

#### Secrets Manager

RDS credential được lưu trong AWS Secrets Manager. Khi chụp screenshot, không hiển thị secret value, password hoặc connection string chứa password.

![RDS secret stored in Secrets Manager with value ](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.4.png)

#### Test kết nối từ EC2/Worker đến RDS

Sau khi kiểm tra Security Group, test kết nối từ EC2 backend hoặc AI worker đến RDS endpoint qua port 5432.

```bash
timeout 5 bash -c '</dev/tcp/<RDS_ENDPOINT>/5432' \
  && echo "RDS port 5432 is reachable" \
  || echo "RDS port 5432 is not reachable"
```

Nếu dùng `psql`, sử dụng  cho thông tin kết nối và không đưa password thật vào markdown.

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=<DB_NAME> user=<DB_USER> sslmode=require"
```

![RDS connectivity test from EC2 or worker](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.5.png)

#### Kiểm tra bảng final alerts

Sau khi kết nối được RDS, kiểm tra schema/table dùng để lưu final alerts. Tên bảng có thể là `final_alerts`, `alerts` hoặc tên tương ứng trong source hiện tại.

```sql
\dt
SELECT COUNT(*) FROM final_alerts;
SELECT id, timestamp, severity, attack_type, source_ip, destination_ip, risk_score
FROM final_alerts
ORDER BY timestamp DESC
LIMIT 5;
```

Nếu table thực tế là `alerts`, thay `final_alerts` bằng `alerts`.

![Final alerts table in PostgreSQL](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.6.png)

#### Kiểm tra worker service

AI Worker chạy dưới dạng systemd service trên EC2 worker instance. Worker đọc message từ SQS, gọi logic AI/Fusion và ghi kết quả vào RDS.

```bash
sudo systemctl status socai-ai-worker --no-pager
```

![AI worker systemd service status](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.7.png)

#### Worker processed message

Worker log chứng minh message đã được consume và xử lý thành công. Evidence quan trọng là dòng `processed message event_id=... alert_id=... elapsed=...`.

```bash
echo "Screenshot: 5.7.8"
sudo journalctl -u socai-ai-worker --no-pager | grep "processed message" | tail -n 5
```

![AI worker processed SQS message](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.8.png)

#### RDS final alert row

Sau khi worker xử lý message, query RDS để xác nhận final alert đã được ghi vào database.

```sql
SELECT id, attack_type, severity, risk_score, technique_id
FROM final_alerts
ORDER BY created_at DESC
LIMIT 5;
```

Nếu table là `alerts`:

```sql
SELECT id, attack_type, severity, risk_score, technique_id
FROM alerts
ORDER BY created_at DESC
LIMIT 5;
```

![Final alert row stored in RDS](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.9.png)

#### API latest alert

Cuối cùng, kiểm tra backend API để xác nhận alert có thể được đọc lại từ RDS. Endpoint dùng trong demo là `/api/alerts/latest`.

```bash
curl -s http://127.0.0.1:8000/api/alerts/latest | python3 -m json.tool
```

![Latest alert returned by backend API](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.10.png)

#### Ghi chú bảo mật

> Không đưa DB password, secret value, access key, `.pem`, token hoặc file `.env` chưa  vào screenshot/workshop.

#### Troubleshooting

| Lỗi | Dấu hiệu | Cách xử lý |
|---|---|---|
| RDS connect timeout | Không kết nối được port 5432 | Kiểm tra RDS Security Group, backend/worker Security Group và subnet route |
| Auth DB fail | `password authentication failed` | Kiểm tra Secrets Manager và DB username/password |
| Worker không nhận message | Không có log `processed message` | Kiểm tra queue URL, region, IAM permission và backend đã gửi SQS chưa |
| Worker lỗi Secrets Manager | `AccessDeniedException GetSecretValue` | Gắn permission `secretsmanager:GetSecretValue` cho worker role |
| Worker lỗi RDS timeout | `ConnectionTimeout` | Kiểm tra RDS Security Group inbound từ worker Security Group và network path |
| RDS không có row mới | Worker processed nhưng DB trống | Kiểm tra insert SQL, transaction commit và table name |
| API không trả alert | `/api/alerts/latest` lỗi hoặc empty | Kiểm tra backend service, DB connection và query latest alert |

Trong section này, bạn đã xác nhận được pipeline backend từ SQS đến AI Worker, RDS PostgreSQL và API latest alert. Phần này chỉ claim Single-AZ/private access và không claim RDS Multi-AZ hoặc production high availability.
