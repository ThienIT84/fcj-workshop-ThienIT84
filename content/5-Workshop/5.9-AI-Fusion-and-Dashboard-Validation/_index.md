---
title : "AI/Fusion and Dashboard Validation"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

# AI/Fusion and Dashboard Validation

## Objective

This section validates the AI/Fusion contract and the SOC dashboard experience. The AI subsections focus on model output and Fusion decision shape, while the dashboard evidence confirms that the analyst UI can display status, KPIs, alerts, MITRE mapping, and event detail.

Current dashboard screenshots may use mock or replay data. They prove UI readiness and workflow shape, not live telemetry, unless the screenshot and surrounding text explicitly say real backend evidence is available.

```text
SQS / S3 evidence
-> Worker / AI models
-> Fusion decision
-> API or WebSocket
-> Dashboard
```

| Area | Current purpose |
|---|---|
| AI1 contract | Define input/output shape and smoke output |
| Fusion schema | Explain how AI and evidence become a final alert |
| Dashboard status | Show mock or real API state clearly |
| Validation limits | Separate demo evidence from production claims |

