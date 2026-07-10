---
title : "SQS and DLQ Setup"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---


## Objective

Create and verify the SQS main queue, queue configuration, DLQ, and redrive policy used by the asynchronous pipeline.

## Step 1: Verify the main queue

Open Amazon SQS and select `socai-dev-normalized-zeek-events-queue`. Confirm the queue name, type, region, URL, and ARN.

![SQS main queue detail](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-01-sqs-main-queue-detail.png)

## Step 2: Review queue configuration

Check the configuration values that affect runtime behavior:

- Visibility timeout controls how long a received message is hidden before retry.
- Message retention controls how long SQS keeps messages that are not deleted.
- Encryption should remain enabled for the queue.

![SQS main queue configuration](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-02-sqs-main-configuration.png)

## Step 3: Verify the DLQ

Open the DLQ `socai-dev-normalized-zeek-events-dlq`. This queue stores messages that fail processing after the configured retry limit.

![SQS DLQ detail](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-03-sqs-dlq-detail.png)

## Step 4: Verify redrive policy

Return to the main queue and confirm that the redrive policy points to the DLQ. The max receive count should be documented so the worker behavior is predictable.

![SQS redrive policy](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-04-sqs-redrive-policy.png)
