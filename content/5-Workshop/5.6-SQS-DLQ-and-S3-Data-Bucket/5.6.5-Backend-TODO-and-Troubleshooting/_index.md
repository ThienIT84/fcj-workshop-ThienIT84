---
title : "Backend Evidence and Troubleshooting"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.6.5. </b> "
---


## Objective

Document the backend integration evidence that has been captured for the SQS path, then keep the remaining S3 object evidence explicit. This prevents the workshop from claiming more than the screenshots prove.

## Evidence plan

| Screenshot | How to use it |
|---|---|
| `w-tin-14-async-api-response-queued.png` | Use as worker/backend follow-up evidence. The visible log shows the worker service is active, but also shows a Secrets Manager `AccessDenied` error, so it belongs in troubleshooting as well. |
| `w-tin-15-sqs-message-after-backend.png` | Use as SQS evidence. It shows a received message body with a normalized HTTP event after backend ingestion. |
| `w-tin-16-s3-object-after-backend.png` | Not available yet. Do not reference it and do not claim backend `put_object` evidence until this screenshot exists. |
| `w-tin-17-backend-sqs-s3-log.png` | Use as backend API evidence. It shows `POST /api/events/http/async` returning `202 Accepted` for the evidence event. |

## Backend API accepted the event

This log proves that the backend accepted the async HTTP event request and returned `202 Accepted`. It is evidence for the ingestion endpoint, not by itself proof that every downstream step completed.

![Backend async API accepted event](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-17-backend-sqs-s3-log.png)

## SQS message created after backend ingestion

This screenshot shows the message body received from SQS. The event includes a schema version, event type, event ID, timestamp, source and destination IP fields, and HTTP evidence. This supports the claim that backend ingestion can create a queue message.

![SQS message after backend ingestion](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-15-sqs-message-after-backend.png)

## Worker follow-up and current blocker

The worker service is enabled and active, but the captured log shows `AccessDeniedException` for `secretsmanager:GetSecretValue`. This is useful troubleshooting evidence: the SQS path has evidence, but worker-to-RDS processing still needs the correct Secrets Manager permission before the end-to-end pipeline can be claimed as complete.

![Worker log and Secrets Manager access issue](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-14-async-api-response-queued.png)

## Remaining evidence

The S3 object evidence screenshot is still missing. Until `w-tin-16-s3-object-after-backend.png` exists, keep backend `put_object` validation as pending.

## Validation checklist

| Check | Status |
|---|---|
| Main queue exists | Covered in 5.6.1 |
| DLQ exists | Covered in 5.6.1 |
| S3 Data Bucket exists | Covered in 5.6.2 |
| IAM policies are prepared | Covered in 5.6.3 |
| Manual SQS/S3 validation works | Covered in 5.6.4 |
| Backend async endpoint accepts event | Evidence captured in `w-tin-17` |
| Backend-created SQS message exists | Evidence captured in `w-tin-15` |
| Backend-created S3 object exists | Pending; screenshot `w-tin-16` not available |
| Worker can read DB secret | Blocked by `secretsmanager:GetSecretValue` AccessDenied in `w-tin-14` |

## Troubleshooting

| Symptom | Check |
|---|---|
| Backend gets AccessDenied for SQS | Verify `socai-dev-backend-role` has `sqs:SendMessage` |
| Backend gets AccessDenied for S3 | Verify `socai-dev-backend-role` has `s3:PutObject` for the data bucket |
| Queue is not receiving messages | Check queue URL, region, and backend logs |
| Worker cannot load DB secret | Add `secretsmanager:GetSecretValue` permission for the worker role and verify the secret ARN |
| Worker retries messages | Check visibility timeout, worker logs, and `DeleteMessage` |
| Messages enter the DLQ | Inspect failed payloads and worker errors before replay |

