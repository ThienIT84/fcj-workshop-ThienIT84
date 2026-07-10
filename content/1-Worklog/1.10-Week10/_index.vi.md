---
title: "Tuần 10 - Bàn giao Model AI, Backend API & Fusion Layer"
date: 2026-06-22
weight: 10
chapter: false
draft: false
pre: " <b> 1.10. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Trọng tâm chính | Tích hợp backend API và đa model |
| Mục tiêu chính | Tập hợp các AI artifact thành API contract, adapter, replay path và logic fusion |
| Framework backend | FastAPI |
| Endpoint chính | `POST /api/events` |
| Định tuyến AI | Flow evidence → AI1/AI2A; HTTP evidence → AI2B |
| Đầu ra Fusion | `attack_type`, `risk_score`, `severity`, contributors, excluded models |
| Triển khai AWS | Chuyển sang Tuần 11 |
| Trạng thái tuần | Hoàn thiện dạng docs-first; ảnh chụp màn hình có thể bổ sung sau |

## Ghi chú nguồn bằng chứng

Tuần 10 sử dụng bằng chứng backend và tích hợp từ các đường dẫn dự án:

```text
backend/
docs/backend_integration_contract.md
docs/multi_model_fusion_mvp_code_reading_guide.md
```

Trang Hugo công khai không đăng tải file code thô, model binary, file môi trường, đường dẫn tuyệt đối nội bộ hoặc thông tin xác thực.

Ghi chú nội bộ Tuần 10 được lưu tại:

```text
docs/worklog/week10/
```

Ảnh chụp màn hình tùy chọn có thể bổ sung sau tại:

```text
static/images/worklog/week10/
```

## Mục tiêu

Mục tiêu của Tuần 10 là chuyển từ bằng chứng dataset và model-readiness sang tích hợp backend.

Tuần 8 đã tổ chức workspace dataset. Tuần 9 rà soát QA và ranh giới model-readiness. Tuần 10 tập trung vào cách các AI artifact đã chuẩn bị kết nối với SOC backend:

```text
AI artifacts
-> backend adapters
-> API contracts
-> fusion service
-> final alert DTO
-> dashboard / WebSocket path
```

Tuần 10 không re-train lại model. Tuần này tập hợp các artifact đã huấn luyện hoặc đã sẵn sàng vào backend contract, adapter thực tế, replay/tailer script và một fusion layer dựa trên rule.

## Bối cảnh dự án

Dự án Hybrid IDS/AI SOC cần nhiều hơn là các báo cáo AI riêng lẻ. Một SOC MVP hữu ích cần một API layer có thể nhận event, định tuyến bằng chứng đến đúng thành phần AI và trả về một alert object nhất quán.

Backend MVP vì vậy tách biệt các trách nhiệm:

- API route tiếp nhận security event đã được chuẩn hóa.
- Orchestrator quyết định AI adapter nào áp dụng cho mỗi event.
- AI adapter trả về model output chuẩn hóa.
- Fusion service chuyển đổi AI output và rule evidence thành alert cuối cùng.
- Store và WebSocket layer hiển thị alert cho dashboard.

Tuần này là cầu nối giữa công việc AI và triển khai cloud. Việc chuyển lên AWS runtime được cố ý để sang Tuần 11.

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày | Công việc đã thực hiện | Kết quả | Vấn đề / quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Rà soát cấu trúc Backend MVP | 22/06/2026 | Rà soát cấu trúc app FastAPI, routes, in-memory store, WebSocket manager và README backend | Các entry point và runtime mode chính của backend được xác định | Giữ Tuần 10 tập trung vào tích hợp backend/API, không phải AWS migration | Rà soát API contract |
| Rà soát API contract | 23/06/2026 | Rà soát event envelope, HTTP wrapper endpoint, model output contract và final alert DTO | `POST /api/events` trở thành ingestion contract chính | Giữ một final alert DTO thay vì các biến thể schema song song | Rà soát thiết kế adapter |
| Rà soát Model adapter | 25/06/2026 | Rà soát hành vi adapter AI1, AI2A, AI2B, mock, unavailable và base | Pattern `supports -> build_input -> predict` được ghi lại | Adapter output phải phân biệt `not_applicable` với `not_available` | Rà soát hành vi fusion |
| Rà soát Fusion layer | 27/06/2026 | Rà soát fusion rule, contributors, excluded models, severity, risk score và ánh xạ `attack_type` cuối cùng | Fusion tạo ra quyết định alert nhất quán từ AI output đầy đủ hoặc một phần | Dashboard phải dùng `attack_type` top-level cuối cùng, không dùng raw AI2B label | Rà soát replay và live path |
| Rà soát Replay và tailer path | 28/06/2026 | Rà soát replay script cục bộ, Zeek `conn.log` tailer, Zeek `http.log` tailer và AI2A feature enrichment path | Các replay/live ingestion path được ánh xạ cho lab telemetry cục bộ | Live path cần runtime evidence trước khi trình bày như một deployment vận hành hoàn chỉnh | Hợp nhất validation matrix |
| Hợp nhất validation và evidence | 28/06/2026 | Rà soát backend test và tạo docs Tuần 10 cho API contract, adapter, fusion, replay và ranh giới | Tuần 10 sẵn sàng như một worklog docs-first | Ảnh chụp màn hình có thể bổ sung sau nếu cần | Dùng Tuần 11 cho AWS MVP deployment |

## Tổng quan kiến trúc Backend

Backend MVP tuân theo luồng sau:

```text
POST /api/events
-> normalize event envelope
-> EventOrchestrator
-> AI1 / AI2A / AI2B adapters khi được hỗ trợ
-> FusionService
-> final alert DTO
-> in-memory alert store
-> WebSocket alert.created
-> frontend dashboard
```

Thiết kế quan trọng là backend không ép mọi event qua mọi model. Mỗi model chỉ nhận loại evidence mà nó hỗ trợ.

| Evidence của event | AI path |
| --- | --- |
| `evidence.flow` | AI1 và AI2A |
| `evidence.http` | AI2B |
| `evidence.suricata` | Fusion rule evidence, adapter path trong tương lai |
| Combined evidence | Nhiều adapter có thể chạy |

## API Contract

Các endpoint chính được rà soát trong Tuần 10:

| Endpoint | Mục đích |
| --- | --- |
| `GET /health` | Kiểm tra sức khỏe runtime |
| `POST /api/events` | Endpoint tiếp nhận event chính |
| `POST /api/events/http` | Wrapper tiện lợi cho HTTP-only event |
| `GET /api/alerts` | Liệt kê alert DTO từ in-memory store |
| `GET /api/summary` | Trả về summary metric từ store |
| `POST /api/replay/demo` | Tạo demo event |
| `WS /ws/alerts` | Broadcast dữ liệu ban đầu và alert mới |

Final alert DTO hiển thị các trường như:

```text
attack_type
severity
risk_score
detected_by
zeek_evidence
suricata_evidence
ai_analysis.ai1
ai_analysis.ai2a
ai_analysis.ai2b
ai_analysis.fusion
```

Tuần 10 giữ nguyên hình dạng DTO hiện tại. Không đưa vào response schema song song.

## Thiết kế Model Adapter

Tất cả adapter đều tuân theo cùng interface khái niệm:

```text
supports(event)
-> build_input(event)
-> predict(model_input)
```

Trạng thái model output là một phần của contract:

| Trạng thái | Ý nghĩa |
| --- | --- |
| `completed` | Adapter đã chạy và tạo ra dự đoán hợp lệ |
| `not_applicable` | Event không chứa evidence phù hợp cho model đó |
| `not_available` | Evidence tương thích nhưng model artifact/config không khả dụng |
| `failed` | Adapter được gọi nhưng inference hoặc preprocessing thất bại |
| `simulated` | Mock/demo output có chủ đích |

Sự phân biệt này quan trọng vì một model không áp dụng cho event không nên bị coi là lỗi model.

## Ranh giới tích hợp AI1

AI1 là anomaly detector ở cấp độ flow.

| Hạng mục | Kết quả Tuần 10 |
| --- | --- |
| Adapter | `ai1_real.py` |
| Phạm vi evidence | `evidence.flow` |
| Input đã chuẩn bị dự kiến | `evidence.flow.ai1_features` |
| Nhãn output | `NORMAL` hoặc `ANOMALY` |
| Hành vi khi thiếu artifact | `not_available` |
| Hành vi khi thiếu feature vector | `not_available` kèm lý do |

AI1 sẵn sàng tích hợp như một artifact contract, nhưng backend vẫn cần AI1 feature đã chuẩn bị trước khi có thể chạy inference thực sự. Adapter không nên suy diễn giá trị feature ẩn từ một flow object tối giản.

## Ranh giới tích hợp AI2A

AI2A là supervised attack classifier ở cấp độ flow.

| Hạng mục | Kết quả Tuần 10 |
| --- | --- |
| Adapter | `ai2a_real.py` |
| Phạm vi evidence | Zeek flow evidence |
| Candidate | `rf_v2_1_full_safe_plus_ssh_minimal` |
| Ngưỡng frozen | `0.9` |
| Feature vector | 41 feature đã frozen |
| Hành vi dưới ngưỡng | Nhãn trở thành `unknown` |
| Hành vi thiếu feature | `not_available`, không đoán |

Replay bridge và `conn.log` tailer có thể làm giàu các Zeek flow row đã chuẩn hóa thành frozen AI2A feature vector trước khi post lên backend. Một flow payload viết tay nhỏ không có frozen vector là chưa đủ cho AI2A inference thực sự.

## Ranh giới tích hợp AI2B

AI2B là HTTP semantic detector.

| Hạng mục | Kết quả Tuần 10 |
| --- | --- |
| Adapter | `ai2b_real.py` |
| Phạm vi evidence | HTTP method và URI |
| Nhãn | `NONE`, `SQLI`, `XSS` |
| Ngữ cảnh model | Đường dẫn release-candidate V1.4.9 |
| Output | Xác suất và nhãn semantic |

AI2B áp dụng cho HTTP event. HTTP-only event không nên ép AI1 hoặc AI2A chạy; các model đó được đánh dấu `not_applicable` một cách đúng đắn.

## Fusion Layer

Fusion dựa trên rule trong MVP. Nó chuyển đổi model output thành một quyết định alert cuối cùng.

Hành vi hiện tại:

| Điều kiện | Diễn giải alert cuối cùng |
| --- | --- |
| AI2B `SQLI` | SQL Injection |
| AI2B `XSS` | Cross-Site Scripting |
| AI1 `ANOMALY` và AI2A không normal | Suspicious Network Activity |
| Chỉ AI1 `ANOMALY` | Network Anomaly |
| Không có tín hiệu AI/rule nào được xác nhận | Benign / No Confirmed Attack |

Fusion cũng ghi lại:

```text
confidence_score
risk_score
mode
contributors
excluded_models
decision_version
```

Rule quan trọng:

```text
Dashboard nên dùng attack_type top-level cuối cùng,
không dùng raw AI2B label riêng lẻ.
```

## Replay và Live Tailer Path

Tuần 10 rà soát cầu nối giữa log lab cục bộ và backend API.

| Script | Mục đích |
| --- | --- |
| `backend/scripts/replay_local_lab_logs.py` | Replay Zeek log cục bộ qua `POST /api/events` |
| `backend/scripts/tail_zeek_conn_to_backend.py` | Stream các Zeek `conn.log` row vào backend flow event |
| `backend/scripts/tail_zeek_http_to_backend.py` | Stream các Zeek `http.log` row vào backend HTTP event |
| `backend/scripts/tail_zeek_correlated_to_backend.py` | Đường dẫn correlated stream nâng cao/tương lai |
| `backend/scripts/replay_demo.py` | Tạo demo event |

Thiết kế replay/tailer có giá trị vì nó kết nối Zeek telemetry Tuần 7 và AI dataset Tuần 8/9 với backend API.

## Validation Matrix

| Mảng | Bằng chứng | Trạng thái | Diễn giải Tuần 10 |
| --- | --- | --- | --- |
| FastAPI routes | `backend/app/main.py` | Đã triển khai | Các endpoint API tồn tại |
| API contract | Backend integration contract | Đã triển khai | Schema event và alert được ghi lại |
| Adapter pattern | Base, mock, real, unavailable adapters | Đã triển khai | Định tuyến model rõ ràng |
| AI1 real adapter | `ai1_real.py`, adapter tests | Một phần / artifact contract | Feature đã chuẩn bị là bắt buộc |
| AI2A real adapter | `ai2a_real.py`, replay tests | Đã triển khai kèm ranh giới | Frozen feature vector là bắt buộc |
| AI2B real adapter | `ai2b_real.py` | Đã triển khai với phạm vi HTTP | Áp dụng cho HTTP semantic evidence |
| Fusion | `fusion.py`, fusion tests | Đã triển khai | Quyết định alert cuối dựa trên rule |
| Replay/tailer scripts | backend scripts | Đã triển khai / cần runtime screenshot | Log lab cục bộ có thể định tuyến đến API |

## Vấn đề và Giới hạn

| Giới hạn | Tác động | Follow-up |
| --- | --- | --- |
| Chưa gắn ảnh chụp màn hình | Tuần 10 hiện là docs-first | Thêm ảnh health/API/test sau |
| Live tailing cần runtime evidence | Worklog không nên claim triển khai live operation đầy đủ | Chụp dry-run hoặc live-tail output nếu cần |
| AI1 cần feature vector đã chuẩn bị | Flow evidence tối giản không thể chạy AI1 thực sự | Thêm extraction layer trước khi claim hỗ trợ replay đầy đủ |
| AWS runtime không thuộc Tuần 10 | Cloud migration vẫn tách biệt | Dùng Tuần 11 cho EC2/backend deployment |

## Bài học kinh nghiệm

Bài học chính từ Tuần 10 là tích hợp model là vấn đề về contract, không chỉ là vấn đề về model. Một backend phải xác định khi nào model áp dụng, điều gì xảy ra khi thiếu artifact, cách biểu diễn dự đoán và cách nhiều kết quả một phần trở thành một SOC alert.

Bài học khác là các degraded mode trung thực có giá trị. Một hệ thống vẫn có thể tạo ra alert hữu ích khi chỉ AI2B áp dụng cho HTTP event hoặc khi AI1/AI2A không áp dụng. Điều đó tốt hơn việc ép dự đoán giả để mọi model đều hiển thị đang hoạt động.

## Kết quả trong tuần

Tuần 10 đã thiết lập câu chuyện tích hợp backend cho Hybrid IDS/AI SOC MVP. Dự án hiện có API contract rõ ràng, model adapter pattern, fusion rule layer và replay/tailer path từ Zeek evidence đến backend alert.

Đây hoàn thành quá trình bàn giao từ công việc dataset/model-readiness sang tích hợp ứng dụng. Bước tiếp theo là triển khai MVP runtime trên AWS theo cách tiết kiệm chi phí.

## Kế hoạch tuần tới

Tuần 11 sẽ tập trung vào AWS MVP migration và triển khai runtime:

- chuẩn bị môi trường EC2 runtime,
- đóng gói và chạy backend,
- xác thực `GET /health` và hành vi API trên AWS,
- ghi lại runtime version và các bước triển khai,
- kết nối migration với nền tảng AWS được thiết lập trước đó.
