---
title : "Security, Lifecycle and IAM"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---


## Objective

Verify S3 bucket security, lifecycle/cost-control status, and IAM permissions for the backend and worker roles.

## Step 1: Verify bucket security

The S3 Data Bucket contains security evidence, so Block Public Access must remain enabled and default encryption should be active.

![S3 security settings](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-06-s3-security-settings.png)

## Step 2: Review lifecycle and cost-control status

Lifecycle rules can help expire temporary raw evidence or move older objects to cheaper storage classes. If the rule is not fully configured, keep this as a cost-control .

![S3 lifecycle or ](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-09-s3-lifecycle-or.png)

## Step 3: Verify backend IAM permissions

The backend role `socai-dev-backend-role` should be allowed to send messages to the SQS main queue and write objects to the S3 Data Bucket.

![Backend IAM policy for SQS and S3](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-10-iam-policy-backend-sqs-s3.png)

## Step 4: Verify worker IAM permissions

The worker role `socai-dev-ai-worker-role` should be allowed to receive/delete SQS messages and read required S3 objects.

![Worker IAM policy for SQS and S3](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-11-iam-policy-worker-sqs-s3.png)
