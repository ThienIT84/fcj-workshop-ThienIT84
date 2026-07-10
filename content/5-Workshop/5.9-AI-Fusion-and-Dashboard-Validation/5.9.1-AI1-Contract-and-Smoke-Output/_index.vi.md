---
title : "AI1 contract và smoke output"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.9.1. </b> "
---


## Mục tiêu

Định nghĩa AI1 input/output contract và hiển thị smoke output evidence hiện có.

## Vị trí AI1 trong pipeline

```text
SQS message / S3 evidence
-> Worker
-> AI1 Anomaly Detection
-> Fusion Layer
```

## AI1 input contract

AI1 nên nhận normalized network features như event ID, source, log type, duration, byte counts và connection state.

## AI1 output contract

AI1 nên trả về model name, input type, prediction, anomaly score, confidence score và các features được dùng cho decision.

## Smoke output evidence

Screenshot dưới đây chứng minh shape của example output. Ảnh này không claim production model integration.

![AI1 output example](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-tin-20-ai1-output-example.png)
