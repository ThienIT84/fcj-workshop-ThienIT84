---
title : "Validation Checklist và Limits"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.9.4. </b> "
---

## Dashboard validation

Dashboard overview xác nhận giao diện SOC chính load được và hiển thị các khu vực workflow cho analyst.

![Dashboard overview](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-48-dashboard-overview.png)

KPI và visual analytics evidence validate summary cards và chart layout.

![Dashboard KPI detail](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-49-dashboard-kpi-detail.png)

![Dashboard visual analytics](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-50-dashboard-visual-analytics.png)

Alert table thể hiện workflow dạng danh sách cho analyst. Nếu dữ liệu là mock hoặc replay, wording phải ghi rõ.

![Dashboard alert table](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-51-dashboard-alert-table.png)

MITRE và event detail evidence cho thấy alert có thể được enrich bằng investigation context.

![Dashboard MITRE badge](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-52-dashboard-mitre-badge.png)

![Dashboard event detail evidence](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-53-dashboard-event-detail-evidence.png)

Decision flow view giải thích cách AI và evidence được trình bày cho analyst.

![Dashboard decision flow](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-54-dashboard-decision-flow.png)

Nếu dùng real backend alert evidence, phần mô tả phải ghi chính xác path đã validate.

![Dashboard real alert after backend](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-phu-56-dashboard-real-alert-after-backend.png)

## Giới hạn

- Không claim live telemetry từ screenshot mock hoặc replay.
- Không claim AI2A/AI2B production integration nếu chưa có model và backend evidence.
- Không claim WebSocket readiness nếu chưa có screenshot WebSocket status.

