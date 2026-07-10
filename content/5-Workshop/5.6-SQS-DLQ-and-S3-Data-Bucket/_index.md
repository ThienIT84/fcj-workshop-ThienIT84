---
title : "SQS, DLQ and S3 Data Bucket"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---


## Overview

This section prepares the asynchronous data layer for the FCAJ Hybrid SOC/AWS pipeline. Amazon SQS decouples backend ingestion from worker processing, while the S3 Data Bucket stores raw evidence, normalized events, reports, and exported artifacts.

## Data flow

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

## Resource names

| Resource | Name |
|---|---|
| SQS main queue | `socai-dev-normalized-zeek-events-queue` |
| SQS DLQ | `socai-dev-normalized-zeek-events-dlq` |
| S3 Data Bucket | `socai-dev-data-************` |
| Backend role | `socai-dev-backend-role` |
| Worker role | `socai-dev-ai-worker-role` |

## Subsections

1. [SQS and DLQ Setup](5.6.1-SQS-and-DLQ-Setup/)
2. [S3 Data Bucket and Prefixes](5.6.2-S3-Data-Bucket-and-Prefixes/)
3. [Security, Lifecycle and IAM](5.6.3-Security-Lifecycle-and-IAM/)
4. [Manual SQS/S3 Validation](5.6.4-Manual-SQS-S3-Validation/)
5. [Backend TODO and Troubleshooting](5.6.5-Backend-TODO-and-Troubleshooting/)
