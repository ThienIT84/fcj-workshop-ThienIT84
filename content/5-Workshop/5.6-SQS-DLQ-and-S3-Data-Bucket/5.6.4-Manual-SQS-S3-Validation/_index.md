---
title : "Manual SQS/S3 Validation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---


## Objective

Validate SQS and S3 manually before backend integration evidence is available.

## Step 1: Send a manual SQS message

Use the SQS console or CloudShell to send a test message to `socai-dev-normalized-zeek-events-queue`. This proves the queue can accept messages, but it does not prove backend integration.

![Manual SQS test](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-12-sqs-manual-test.png)

## Step 2: Upload a manual S3 object

Upload a test object to the expected Zeek HTTP prefix in `socai-dev-data-************`. This proves the bucket and prefix are usable, but it does not prove the backend has called `put_object`.

![Manual S3 object test](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-13-s3-manual-object-test.png)

## Result

Manual validation confirms that the SQS and S3 resources are ready for backend integration. Backend-created evidence remains pending.
