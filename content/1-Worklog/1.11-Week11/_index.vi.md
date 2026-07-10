---
title: "Tuần 11 - Pipeline Alert Bất đồng bộ trên AWS: SQS, Worker, RDS và Idempotency"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Trọng tâm chính | Chuyển xử lý alert từ luồng backend trực tiếp sang pipeline bất đồng bộ trên AWS |
| Dịch vụ AWS | CloudFront, Application Load Balancer, EC2, Amazon SQS, Amazon RDS PostgreSQL, Secrets Manager, CloudWatch Logs |
| Backend API | `POST /api/events/http/async`, `GET /api/alerts/latest` |
| Worker service | `socai-ai-worker` đọc message từ SQS main queue |
| Nơi lưu trữ | Bảng RDS PostgreSQL `final_alerts` |
| Kiểm thử chính | Evidence D5-5 duplicate-event/idempotency với `event_id` do caller cung cấp |
| Trạng thái tuần | Đã triển khai và xác thực bằng evidence lưu lại từ môi trường deploy |

## Mục tiêu

Mục tiêu của Tuần 11 là mở rộng backend SOC AI từ demo request/response trực tiếp thành một pipeline cloud bất đồng bộ. Hệ thống cần nhận HTTP security event qua API đã deploy, đưa event vào Amazon SQS, xử lý bằng worker service, và lưu kết quả alert cuối cùng vào RDS PostgreSQL.

Tuần này tập trung vào backend và data pipeline. Realtime rendering trên dashboard được xem là ranh giới tích hợp tiếp theo, không claim là đã hoàn tất trong tuần này.

## Luồng kiến trúc

Luồng đã được xác thực trong tuần này:

```text
POST /api/events/http/async
-> SQS main queue
-> socai-ai-worker
-> AI/Fusion processing
-> RDS final_alerts
-> GET /api/alerts/latest / dashboard API evidence
```

Backend nhận event qua async API và đưa event vào SQS main queue. Worker đọc message, tái sử dụng luồng model/fusion hiện có, rồi ghi alert cuối cùng vào RDS. Alert đã lưu có thể được truy xuất lại qua backend API.

Vì môi trường AWS đã được cleanup sau đó, trang này sử dụng screenshot và terminal evidence được chụp khi deployment còn hoạt động. Các thông tin nhạy cảm như account ID, username, instance ID hoặc IP nội bộ cần được mask trong bộ ảnh public.

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Rà soát kiến trúc bất đồng bộ | 29/06/2026 | Rà soát luồng mục tiêu từ CloudFront/API ingestion đến SQS, worker, RDS persistence và API cho dashboard | Phạm vi Tuần 11 được thu hẹp vào SQS/RDS persistence và idempotency evidence | Realtime dashboard broadcast được để ngoài phạm vi cho đến khi chứng minh được ranh giới worker-to-UI | Tạo và xác thực queue |
| Xác thực SQS queue và DLQ | 30/06/2026 | Tạo và kiểm tra SQS main queue cùng DLQ cho event Zeek/backend đã chuẩn hóa | Cặp queue tồn tại và có thể dùng làm ranh giới bất đồng bộ giữa API và worker | Message trong DLQ cần được inspect trước khi delete hoặc redrive vì đó là forensic evidence | Nối backend async ingestion vào SQS |
| Backend async ingestion | 01/07/2026 | Nối `POST /api/events/http/async` với SQS producer và kiểm tra response `202 Accepted` qua route đã deploy | Backend nhận event theo kiểu bất đồng bộ thay vì phải hoàn tất alert ngay trong request | API cần giữ nguyên `event_id` do caller gửi để test idempotency | Nối worker vào SQS và RDS |
| Worker, Secrets Manager và RDS | 03/07/2026 | Nối `socai-ai-worker` với SQS, Secrets Manager và RDS; debug IAM và connectivity | Worker có thể đọc event từ queue và ghi final alert vào RDS `final_alerts` | Thiếu quyền `secretsmanager:GetSecretValue` và khác biệt env/PYTHONPATH gây lỗi dễ hiểu nhầm khi recovery | Xác thực alert đã lưu và latest-alert API |
| Alert persistence và API evidence | 04/07/2026 | Query RDS `final_alerts` và kiểm tra `/api/alerts/latest` với các alert SQL Injection và XSS | RDS lưu alert cuối cùng gồm severity, attack type, risk score, model output và fusion decision | Dashboard UI nên đọc từ API/persistence path; in-memory WebSocket state là chưa đủ sau khi worker xử lý | Chạy duplicate-event idempotency evidence |
| Xác thực D5-5 idempotency | 05/07/2026 | Gửi cùng một application-level `event_id` ba lần và kiểm tra row count trong RDS | `final_alerts` giữ một row cho event đó trong khi payload phản ánh trạng thái xử lý mới nhất | SQS Standard có at-least-once delivery, nên application phải chịu được duplicate event | Ghi lại evidence và giới hạn của Tuần 11 |

## Tóm tắt triển khai

Tuần 11 bổ sung lớp xử lý bất đồng bộ xung quanh model và fusion code hiện có. Backend chịu trách nhiệm nhận event và đưa vào queue, còn worker chịu trách nhiệm đọc message, xử lý và ghi bản ghi final alert.

Contract lưu trữ quan trọng là `final_alerts.event_id`. `event_id` do caller gửi được backend giữ nguyên, truyền qua SQS và dùng ở RDS layer để upsert alert. Cách này tránh tạo nhiều final alert rows khi cùng một logical event được xử lý nhiều lần.

| Thành phần | Trách nhiệm trong Tuần 11 |
| --- | --- |
| Backend async API | Nhận event input, giữ `event_id`, đưa vào SQS, trả `202 Accepted` |
| SQS main queue | Buffer normalized event giữa API và worker |
| SQS DLQ | Giữ message lỗi để inspect và recovery |
| Worker | Poll SQS, xử lý event, gọi model/fusion logic, ghi RDS |
| RDS `final_alerts` | Lưu final alert payload với hành vi idempotent theo `event_id` |
| `/api/alerts/latest` | Cung cấp browser/API evidence cho alert đã persist mới nhất |

## Bộ ảnh minh chứng

<figure class="worklog-evidence">
  <img src="/images/worklog/week11/w11-e01-sqs-main-dlq-queues.png" alt="SQS main queue and dead-letter queue overview">
  <figcaption>Hình 1 - Amazon SQS hiển thị Standard main queue và DLQ được dùng bởi pipeline event bất đồng bộ.</figcaption>
</figure>

**Quan sát:** Cặp queue tạo ranh giới bất đồng bộ giữa API ingestion và worker processing. Message xuất hiện trong DLQ được giữ để kiểm tra lỗi trước bất kỳ quyết định cleanup nào.

<figure class="worklog-evidence">
  <img src="/images/worklog/week11/w11-e02-rds-final-alerts-query.png" alt="RDS final_alerts query result">
  <figcaption>Hình 2 - RDS `final_alerts` chứa các alert SQL Injection và XSS đã được lưu từ pipeline deploy.</figcaption>
</figure>

**Quan sát:** Query `final_alerts` trả về các row có `event_id`, `severity`, `attack_type`, `risk_score` và `created_at`. Điều này xác nhận output của worker đã được persist ngoài in-memory store của FastAPI.

<figure class="worklog-evidence">
  <img src="/images/worklog/week11/w11-e04-ai2b-fusion-payload.png" alt="AI2B and Fusion SQL Injection decision payload">
  <figcaption>Hình 3 - Payload AI2B và Fusion đã chụp phân loại HTTP request là SQL Injection và tạo final critical decision.</figcaption>
</figure>

**Ghi chú evidence:** Screenshot D5-5 idempotency count và backend log `202 Accepted` không còn được lưu thành file public trước khi môi trường AWS được cleanup. Kết quả test vẫn được ghi trong phần bên dưới, nhưng trang này không trình bày hai screenshot đó như evidence đã publish.

## Xác thực D5-5 Idempotency

D5-5 test cố ý gửi cùng một application-level `event_id` ba lần qua `POST /api/events/http/async`. Test này không ép SQS redeliver cùng một physical message; thay vào đó, nó kiểm tra contract quan trọng với SQS Standard: hệ thống phải an toàn khi cùng một logical event xuất hiện nhiều lần.

Evidence acceptance từ run:

| Kiểm tra | Kết quả |
| --- | --- |
| Cùng một `EVENT_ID` được gửi ba lần | Pass |
| Backend giữ nguyên `EVENT_ID` do caller gửi | Pass |
| RDS count theo `EVENT_ID` vẫn là `1` | Pass |
| Payload đã lưu phản ánh duplicate attempt mới nhất | Pass |
| Worker không tạo duplicate rows trong `final_alerts` | Pass |
| `/api/alerts/latest` khớp với test event trong lúc chạy | Pass |

Source of truth là RDS:

```text
final_alerts.event_id UNIQUE
ON CONFLICT(event_id) DO UPDATE
SELECT count(*) WHERE event_id = EVENT_ID -> 1
```

## Vấn đề và cách xử lý

Vấn đề chính khi deploy không nằm ở model logic mà nằm ở runtime wiring. Ban đầu worker không đọc được RDS secret vì IAM role thiếu quyền `secretsmanager:GetSecretValue` cho `socai-dev/rds/app`. Sau khi sửa permission cho role, các bước kiểm tra tiếp theo tập trung vào RDS connectivity, Security Group, environment variables và chạy Python với đúng `PYTHONPATH`.

RDS access đã được xác thực từ worker/backend host, và service environment được kiểm tra có AWS Region, SQS queue URL và RDS secret ID cần thiết. Sau đó worker có thể xử lý message và ghi final alert.

## Giới hạn còn lại

- Worker-to-dashboard realtime WebSocket broadcast chưa được chứng minh trong Tuần 11.
- Worker đã persist alert vào RDS, nhưng FastAPI WebSocket manager là process-local, nên UI realtime behavior cần validation riêng.
- Dashboard rendering chỉ nên claim ở Week 12 hoặc final demo nếu có evidence riêng.
- Các screenshot là archived evidence từ trước cleanup và cần được mask trước khi public.

## Kết quả Tuần 11

Tuần 11 hoàn thành phần lõi của async backend pipeline cho SOC AI platform. Event có thể được nhận qua async API đã deploy, đưa vào SQS, xử lý bởi worker, lưu vào RDS và truy xuất qua latest-alert API. D5-5 duplicate-event test cung cấp evidence mạnh rằng lớp persistence final alert an toàn trước duplicate logical events.
