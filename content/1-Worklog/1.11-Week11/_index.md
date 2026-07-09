---
title: "Week 11 - AWS Async Alert Pipeline: SQS, Worker, RDS and Idempotency"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary focus | Move alert processing from a direct backend path to an asynchronous AWS pipeline |
| AWS services | CloudFront, Application Load Balancer, EC2, Amazon SQS, Amazon RDS PostgreSQL, Secrets Manager, CloudWatch Logs |
| Backend API | `POST /api/events/http/async`, `GET /api/alerts/latest` |
| Worker service | `socai-ai-worker` consuming messages from the SQS main queue |
| Persistence target | RDS PostgreSQL table `final_alerts` |
| Main validation | D5-5 duplicate-event/idempotency evidence using a caller-provided `event_id` |
| Week status | Implemented and validated with archived deployment evidence |

## Objective

The objective of Week 11 was to extend the SOC AI backend from a direct request/response demo into an asynchronous cloud pipeline. The goal was to accept HTTP security events through the deployed API, queue them in Amazon SQS, process them with a worker service, and persist the final alert result in RDS PostgreSQL.

This week focused on the backend and data pipeline. Dashboard realtime rendering was treated as a downstream integration boundary instead of being claimed as completed in this week.

## Architecture Flow

The deployed flow validated in this week was:

```text
POST /api/events/http/async
-> SQS main queue
-> socai-ai-worker
-> AI/Fusion processing
-> RDS final_alerts
-> GET /api/alerts/latest / dashboard API evidence
```

The backend accepted events through the async API and placed them on the SQS main queue. The worker consumed the messages, reused the existing model/fusion path, and wrote the resulting alert to RDS. The latest persisted alert could then be retrieved through the backend API.

Because the AWS environment was later cleaned up, this page uses the screenshots and terminal evidence captured while the deployment was still active. Sensitive account and identity details should be masked in the public image set before publishing.

## Daily Worklog

| Activity | Date | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- |
| Async architecture review | 29/06/2026 | Reviewed the target flow from CloudFront/API ingestion to SQS, worker processing, RDS persistence, and dashboard API retrieval | The Week 11 scope was narrowed to SQS/RDS persistence and idempotency evidence | Realtime dashboard broadcast was kept out of scope until the worker-to-UI boundary could be proven | Create and validate the queues |
| SQS queue and DLQ validation | 30/06/2026 | Created and inspected the SQS main queue and DLQ for normalized Zeek/backend events | The queue pair existed and could be used as the async boundary between API and worker | DLQ messages had to be inspected before deleting or redriving because they are forensic evidence | Connect backend async ingestion to SQS |
| Backend async ingestion | 01/07/2026 | Connected `POST /api/events/http/async` to the SQS producer and verified `202 Accepted` responses through the deployed route | Backend accepted events asynchronously instead of requiring synchronous alert completion | The API had to preserve caller-provided `event_id` for idempotency tests | Connect the worker to SQS and RDS |
| Worker, Secrets Manager, and RDS wiring | 03/07/2026 | Connected `socai-ai-worker` to SQS, Secrets Manager, and RDS; debugged IAM and connectivity issues | Worker could read queued events and write final alerts into RDS `final_alerts` | Missing `secretsmanager:GetSecretValue` and environment/PYTHONPATH differences caused misleading failures during recovery | Validate persisted alerts and latest-alert API |
| Alert persistence and API evidence | 04/07/2026 | Queried RDS `final_alerts` and checked `/api/alerts/latest` for SQL Injection and XSS alert records | RDS stored final alerts with severity, attack type, risk score, model output, and fusion decision data | The dashboard UI should read from the persisted API path; in-memory WebSocket state is not enough after worker processing | Run duplicate-event idempotency evidence |
| D5-5 idempotency validation | 05/07/2026 | Sent the same application-level `event_id` three times and checked the RDS row count | `final_alerts` kept one row for the event while preserving the latest payload state | SQS Standard is at-least-once, so the application must tolerate duplicate event delivery | Document evidence and Week 11 limits |

## Implementation Summary

Week 11 added the asynchronous processing layer around the existing model and fusion code. The backend became responsible for accepting events and queueing them, while the worker became responsible for consuming queued messages and writing final alert records.

The key persistence contract was `final_alerts.event_id`. A caller-provided `event_id` was preserved by the backend, sent through SQS, and used by the RDS layer to upsert the alert. This prevented duplicate final alert rows when the same logical event was processed multiple times.

| Component | Week 11 responsibility |
| --- | --- |
| Backend async API | Accept event input, preserve `event_id`, enqueue to SQS, return `202 Accepted` |
| SQS main queue | Buffer normalized events between API and worker |
| SQS DLQ | Retain failed messages for inspection and recovery |
| Worker | Poll SQS, process the event, call model/fusion logic, write to RDS |
| RDS `final_alerts` | Persist final alert payload with idempotent `event_id` behavior |
| `/api/alerts/latest` | Provide browser/API evidence for the latest persisted alert |

## Evidence Gallery

<figure class="worklog-evidence">
  <img src="/images/worklog/week11/w11-e02-rds-final-alerts-query.png" alt="RDS final_alerts query result">
  <figcaption>Figure 1 - RDS `final_alerts` contained persisted SQL Injection and XSS alerts from the deployed pipeline.</figcaption>
</figure>

**Observation:** Querying `final_alerts` returned alert rows with `event_id`, `severity`, `attack_type`, `risk_score`, and `created_at`. This confirmed that worker output was persisted beyond the in-memory FastAPI store.

**Evidence note:** The SQS queue overview, D5-5 idempotency count, AI2B/Fusion payload, and backend `202 Accepted` log screenshots are referenced in the written validation, but their public image files were not present in this repository path during this cleanup pass. They should be added only after masking and placed under `static/images/worklog/week11/` with the expected `w11-*` filenames.

## D5-5 Idempotency Validation

The D5-5 test intentionally sent the same application-level `event_id` three times through `POST /api/events/http/async`. This did not force SQS to redeliver one physical message; instead, it tested the application contract that matters for SQS Standard semantics: the system must be safe when the same logical event appears more than once.

Acceptance evidence from the run:

| Check | Result |
| --- | --- |
| Same `EVENT_ID` sent three times | Pass |
| Backend preserved caller-provided `EVENT_ID` | Pass |
| RDS count by `EVENT_ID` remained `1` | Pass |
| Stored payload reflected the latest duplicate attempt | Pass |
| Worker did not create duplicate `final_alerts` rows | Pass |
| `/api/alerts/latest` matched the test event during the run | Pass |

The source of truth was RDS:

```text
final_alerts.event_id UNIQUE
ON CONFLICT(event_id) DO UPDATE
SELECT count(*) WHERE event_id = EVENT_ID -> 1
```

## Issues and Fixes

The main deployment issue was not model logic; it was runtime wiring. The worker initially could not read the RDS secret because its IAM role did not have `secretsmanager:GetSecretValue` for `socai-dev/rds/app`. After the role permission was corrected, the next checks focused on RDS connectivity, Security Group rules, environment variables, and running Python with the correct `PYTHONPATH`.

RDS access was validated from the worker/backend host, and the service environment was confirmed to include the required AWS Region, SQS queue URL, and RDS secret ID. After that, the worker could process messages and write final alerts.

## Remaining Limitations

- Worker-to-dashboard realtime WebSocket broadcast was not proven in Week 11.
- The worker persisted alerts in RDS, but the FastAPI WebSocket manager is process-local, so UI realtime behavior requires separate validation.
- Dashboard rendering should be claimed only where there is separate Week 12 or final-demo evidence.
- The screenshots are archived evidence from before cleanup and should be masked before public publication.

## Week 11 Outcome

Week 11 completed the core async backend pipeline for the SOC AI platform. Events could be accepted through the deployed async API, queued in SQS, processed by the worker, persisted in RDS, and retrieved through the latest-alert API. The D5-5 duplicate-event test provided strong evidence that the final alert persistence layer was safe against duplicate logical events.
