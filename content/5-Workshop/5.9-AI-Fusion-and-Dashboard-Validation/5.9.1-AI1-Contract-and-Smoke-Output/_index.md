---
title : "AI1 Contract and Smoke Output"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.9.1. </b> "
---


## Objective

Define the AI1 input/output contract and show the current smoke output evidence.

## AI1 position in the pipeline

```text
SQS message / S3 evidence
-> Worker
-> AI1 Anomaly Detection
-> Fusion Layer
```

## AI1 input contract

AI1 should receive normalized network features such as event ID, source, log type, duration, byte counts, and connection state.

## AI1 output contract

AI1 should return model name, input type, prediction, anomaly score, confidence score, and the features used by the decision.

## Smoke output evidence

The screenshot below proves the example output shape. It does not claim production model integration.

![AI1 output example](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-tin-20-ai1-output-example.png)
