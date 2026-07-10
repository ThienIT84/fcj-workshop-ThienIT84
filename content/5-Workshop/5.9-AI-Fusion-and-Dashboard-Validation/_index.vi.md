---
title : "AI/Fusion và Dashboard Validation"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

# AI/Fusion và Dashboard Validation

## Mục tiêu

Phần này validate AI/Fusion contract và trải nghiệm SOC dashboard. Các subsection AI tập trung vào model output và Fusion decision shape, còn dashboard evidence chứng minh analyst UI hiển thị được status, KPI, alert, MITRE mapping và event detail.

Các screenshot dashboard hiện tại có thể dùng mock hoặc replay data. Chúng chứng minh UI readiness và workflow shape, không phải live telemetry, trừ khi screenshot và phần mô tả nói rõ đã có real backend evidence.

```text
SQS / S3 evidence
-> Worker / AI models
-> Fusion decision
-> API hoặc WebSocket
-> Dashboard
```

| Nhóm | Mục đích hiện tại |
|---|---|
| AI1 contract | Định nghĩa input/output shape và smoke output |
| Fusion schema | Giải thích cách AI và evidence tạo final alert |
| Dashboard status | Hiển thị rõ mock hoặc real API state |
| Validation limits | Tách demo evidence khỏi production claims |

