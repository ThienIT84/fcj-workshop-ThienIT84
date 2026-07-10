---
title : "S3 Data Bucket and Prefixes"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---


## Objective

Verify the S3 Data Bucket and document the prefix convention used for raw evidence, normalized events, reports, and exports.

## Step 1: Verify the data bucket

Open Amazon S3 and select `socai-dev-data-************`. This bucket is for SOC data, not static website hosting.

![S3 data bucket overview](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-05-s3-data-bucket-overview.png)

## Step 2: Review top-level prefixes

Use stable prefixes so backend, workers, and manual validation steps can locate objects consistently:

```text
raw/
normalized/
reports/
exports/
```

![S3 prefix structure](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-07-s3-prefix-structure.png)

## Step 3: Review Zeek prefixes

Zeek evidence should be separated by log type. The expected examples are:

```text
raw/zeek/http/
raw/zeek/conn/
```

![S3 Zeek prefixes](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-08-s3-zeek-prefixes.png)
