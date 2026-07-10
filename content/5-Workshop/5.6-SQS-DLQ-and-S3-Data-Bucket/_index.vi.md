---
title : "SQS, DLQ và S3 Data Bucket"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---


## Tổng quan

Section này chuẩn bị data layer bất đồng bộ cho pipeline FCAJ Hybrid SOC/AWS. Amazon SQS tách backend ingestion khỏi worker processing, còn S3 Data Bucket lưu raw evidence, normalized event, report và exported artifact.

## Luồng dữ liệu

```text
EC2 Backend
-> SQS Main Queue
-> Worker / AI / Fusion
-> RDS final alerts

EC2 Backend
-> S3 Data Bucket
-> raw/zeek/http/
-> raw/zeek/conn/
-> normalized/events/
-> reports/
-> exports/
```

## Tên resource

| Resource | Name |
|---|---|
| SQS main queue | `socai-dev-normalized-zeek-events-queue` |
| SQS DLQ | `socai-dev-normalized-zeek-events-dlq` |
| S3 Data Bucket | `socai-dev-data-************` |
| Backend role | `socai-dev-backend-role` |
| Worker role | `socai-dev-ai-worker-role` |

## Các subsection

1. [SQS and DLQ Setup](5.6.1-SQS-and-DLQ-Setup/)
2. [S3 Data Bucket and Prefixes](5.6.2-S3-Data-Bucket-and-Prefixes/)
3. [Security, Lifecycle and IAM](5.6.3-Security-Lifecycle-and-IAM/)
4. [Manual SQS/S3 Validation](5.6.4-Manual-SQS-S3-Validation/)
5. [Backend TODO and Troubleshooting](5.6.5-Backend-TODO-and-Troubleshooting/)
