---
title : "RDS Secrets Worker Pipeline"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

In this section, you validate the backend storage and processing layer of the AWS pipeline. RDS PostgreSQL stores final alerts after the worker processes messages from SQS. Secrets Manager stores database connection information so credentials do not need to be hard-coded in the source code.

For this workshop, RDS runs in Single-AZ mode to reduce demo cost. Multi-AZ is a production-style extension and is not deployed in the current environment.

#### Processing flow

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

The worker deletes a message from SQS only after the event is processed successfully and the final alert is written to RDS. This helps reduce the risk of losing events if the worker fails in the middle of processing.

#### RDS Single-AZ and private access

RDS PostgreSQL is configured for private access. The database does not allow public access and only accepts PostgreSQL traffic on port 5432 from the backend/worker Security Group. This is an important security baseline so the database is not exposed directly to the Internet.

RDS PostgreSQL is available and running in Single-AZ mode to reduce demo cost.

![RDS PostgreSQL Single-AZ summary](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.1.png)

RDS uses private connectivity, public access is disabled, and the endpoint listens on port 5432.

![RDS private connectivity and endpoint](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.2.png)

The Security Group allows only the backend/worker to access PostgreSQL on port 5432.

![RDS security group inbound rule](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.3.png)

#### Secrets Manager

The RDS credential is stored in AWS Secrets Manager. When capturing screenshots, do not show secret values, passwords, or connection strings that include passwords.

![RDS secret stored in Secrets Manager with value ](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.4.png)

#### Test connectivity from EC2/Worker to RDS

After checking the Security Group, test connectivity from the backend EC2 instance or AI worker to the RDS endpoint on port 5432.

```bash
timeout 5 bash -c '</dev/tcp/<RDS_ENDPOINT>/5432' \
  && echo "RDS port 5432 is reachable" \
  || echo "RDS port 5432 is not reachable"
```

If you use `psql`, keep connection details as s and do not include a real password in the markdown.

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=<DB_NAME> user=<DB_USER> sslmode=require"
```

![RDS connectivity test from EC2 or worker](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.5.png)

#### Check the final alerts table

After connecting to RDS, check the schema/table used to store final alerts. The table name can be `final_alerts`, `alerts`, or the corresponding name used in the current source code.

```sql
\dt
SELECT COUNT(*) FROM final_alerts;
SELECT id, timestamp, severity, attack_type, source_ip, destination_ip, risk_score
FROM final_alerts
ORDER BY timestamp DESC
LIMIT 5;
```

If the actual table is `alerts`, replace `final_alerts` with `alerts`.

![Final alerts table in PostgreSQL](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.6.png)

#### Check the worker service

The AI Worker runs as a systemd service on the EC2 worker instance. The worker reads messages from SQS, runs the AI/Fusion logic, and writes the result to RDS.

```bash
sudo systemctl status socai-ai-worker --no-pager
```

![AI worker systemd service status](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.7.png)

#### Worker processed message

The worker log proves that a message was consumed and processed successfully. The important evidence is a log line similar to `processed message event_id=... alert_id=... elapsed=...`.

```bash
echo "Screenshot: 5.7.8"
sudo journalctl -u socai-ai-worker --no-pager | grep "processed message" | tail -n 5
```

![AI worker processed SQS message](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.8.png)

#### RDS final alert row

After the worker processes a message, query RDS to confirm that the final alert was written to the database.

```sql
SELECT id, attack_type, severity, risk_score, technique_id
FROM final_alerts
ORDER BY created_at DESC
LIMIT 5;
```

If the table is `alerts`:

```sql
SELECT id, attack_type, severity, risk_score, technique_id
FROM alerts
ORDER BY created_at DESC
LIMIT 5;
```

![Final alert row stored in RDS](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.9.png)

#### API latest alert

Finally, check the backend API to confirm that the alert can be read back from RDS. The endpoint used in the demo is `/api/alerts/latest`.

```bash
curl -s http://127.0.0.1:8000/api/alerts/latest | python3 -m json.tool
```

![Latest alert returned by backend API](/images/5-Workshop/5.7-RDS-Secrets-Worker-Pipeline/5.7.10.png)

#### Security note

> Do not include DB passwords, secret values, access keys, `.pem` files, tokens, or un `.env` files in screenshots or workshop content.

#### Troubleshooting

| Issue | Symptom | Resolution |
|---|---|---|
| RDS connect timeout | Cannot connect to port 5432 | Check the RDS Security Group, backend/worker Security Group, and subnet route |
| Database authentication fails | `password authentication failed` | Check Secrets Manager and the database username/password |
| Worker does not receive messages | No `processed message` log | Check the queue URL, region, IAM permissions, and whether the backend has sent messages to SQS |
| Worker cannot read Secrets Manager | `AccessDeniedException GetSecretValue` | Attach `secretsmanager:GetSecretValue` permission to the worker role |
| Worker RDS timeout | `ConnectionTimeout` | Check RDS Security Group inbound rules from the worker Security Group and the network path |
| RDS has no new row | Worker processed the event but the database is empty | Check the insert SQL, transaction commit, and table name |
| API does not return an alert | `/api/alerts/latest` returns an error or empty response | Check the backend service, database connection, and latest alert query |

In this section, you confirmed the backend pipeline from SQS to AI Worker, RDS PostgreSQL, and the latest alert API. This section only claims Single-AZ/private access and does not claim RDS Multi-AZ or production high availability.
