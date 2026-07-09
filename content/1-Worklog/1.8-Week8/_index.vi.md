---
title: "Tuần 8 - Workspace Dataset AI & Phủ sóng Attack Profile"
date: 2026-06-07
weight: 8
chapter: false
draft: false
pre: " <b> 1.8. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Trọng tâm chính | Tổ chức workspace dataset AI |
| Mục tiêu chính | Ánh xạ IDS telemetry cục bộ, normal profile, attack profile và dataset riêng cho từng AI component |
| Nguồn dataset | Artifact Zeek telemetry từ lab IDS cục bộ |
| Đường cơ sở bình thường | Dataset traffic bình thường P01–P12 |
| Các AI component | AI1 anomaly detector, AI2A flow classifier, AI2B HTTP semantic detector |
| Phủ sóng attack chính | A01–A06 và A08–A10 artifact-backed; A07 aggregate-only; A12 HTTP semantic evidence reviewed |
| Khoảng thiếu bằng chứng | A11 không tìm thấy trong lần quét workspace hiện tại |
| Không khẳng định | Hoàn thành A11, sẵn sàng production, hoàn thành training model |
| Trạng thái trong tuần | Triển khai dạng docs-first draft; ảnh chụp màn hình có thể được bổ sung sau nếu cần |

## Lưu ý về Nguồn Bằng chứng

Tuần 8 dựa trên workspace dataset và các report lưu trong các đường dẫn dự án logic như:

```text
Dataset/tools/final_dataset/
Dataset/tools/attack_profiles/
Dataset/tools/ai1_modeling/
Dataset/tools/ai2a_modeling/
Dataset/tools/ai2b_modeling/
```

Trang này không công khai raw report file, CSV file, log, thông tin đăng nhập hay environment file. Các evidence index nội bộ được duy trì trong:

```text
docs/worklog/week8/
```

Ảnh chụp màn hình công khai là tùy chọn trong giai đoạn docs-first drafting. Trước khi xuất bản, nên bổ sung bằng chứng sanitize được chọn lọc để hỗ trợ các claim chính về workspace, metric và coverage. Chỉ file ảnh đã redact mới được đặt trong:

```text
static/images/worklog/week8/
```

## Phương pháp Bằng chứng và Bảng Trạng thái

Tuần 8 review các đường dẫn workspace theo project-relative path, dataset summary, model handoff document, feature manifest, QA report và các bằng chứng raw-log được chọn lọc. Trừ khi được ghi rõ là đã reproduced, các metric trên trang này là artifact-backed value — không phải kết quả được tái tạo độc lập trong phiên worklog Tuần 8.

Các status label sau được sử dụng trong trang này:

| Status | Ý nghĩa |
| --- | --- |
| Reproduced | Lệnh hoặc script đã được chạy lại trong Tuần 8 và tạo ra kết quả mới |
| Artifact-backed | Một report hoặc dataset artifact hiện có hỗ trợ cho khẳng định này |
| Aggregate-only | Bằng chứng chỉ tồn tại trong dataset hoặc report tổng hợp, không có profile summary độc lập |
| Context-only | Reviewed để hiểu kiến trúc hoặc input contract, không trình bày như kết quả triển khai Tuần 8 |
| Not found | Không tìm thấy artifact liên quan trong lần quét workspace hiện tại |
| Deferred | Tài liệu có sẵn được dành cho phân tích sâu hơn của Tuần 9 |

## Mục tiêu

Mục tiêu của Tuần 8 là tổ chức workspace dataset đằng sau dự án Hybrid IDS/AI SOC và giải thích cách các artifact telemetry khác nhau hỗ trợ từng AI component.

Thay vì xử lý công việc dataset như một tác vụ thu thập phẳng đơn lẻ, Tuần 8 tách bằng chứng thành ba trách nhiệm AI:

- AI1 sử dụng Zeek flow feature để phát hiện anomaly.
- AI2A sử dụng Zeek `conn.log`-derived flow feature để phân loại tấn công mạng có giám sát.
- AI2B sử dụng bằng chứng HTTP từ Zeek `http.log` để phân loại HTTP semantic SQLi/XSS.

## Bối cảnh Dự án

Tuần 7 đã xác minh rằng lab cục bộ có thể tạo ra Zeek telemetry. Tuần 8 chuyển từ "lab có thể tạo telemetry không?" sang "telemetry được tổ chức thành dataset cho hệ thống AI như thế nào?"

Workspace dataset kết nối lab IDS cục bộ với SOC MVP về sau:

```text
Traffic IDS cục bộ
-> Zeek conn.log và http.log
-> dataset normal và attack profile
-> input dataset cho AI1 / AI2A / AI2B
-> tích hợp backend và fusion-layer ở các tuần sau
```

Do đó, Tuần 8 tập trung vào coverage dataset và ranh giới bằng chứng, không phải hiệu năng model cuối cùng.

## Kiến trúc Dataset AI

| Component | Input chính | Mục đích | Vị trí trong Tuần 8 |
| --- | --- | --- | --- |
| AI1 | Flow feature từ Zeek `conn.log` | Phát hiện anomaly không giám sát | Reviewed artifact context |
| AI2A | Zeek `conn.log` flow-level feature | Phân loại tấn công mạng đa lớp có giám sát | Reviewed workspace dataset và profile coverage |
| AI2B | Normalized URI/query/request-line evidence từ Zeek `http.log` | Phân loại HTTP semantic cho `NONE`, `SQLI` và `XSS` | Reviewed A12 evidence và workspace dataset |
| Fusion layer | AI output và rule evidence | Final risk score | Hoãn lại cho công việc thiết kế backend/system sau |

Quyết định thiết kế quan trọng là giữ riêng biệt collection evidence, model feature, label và downstream risk scoring. Tuần 8 không khẳng định rằng risk score đã là một phần của dataset.

AI2B tập trung vào normalized URI/query và request-line evidence từ Zeek HTTP telemetry. Tuần 8 không khẳng định HTTP request-body inspection hoặc full application-layer payload coverage.

## Trình tự Xem xét Bằng chứng

| Hoạt động | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Review workspace dataset | 07/06/2026 | Review workspace dataset cục bộ và tách biệt dữ liệu bình thường, attack profile và các folder modeling dành riêng cho từng AI | Workspace dataset được ánh xạ xung quanh trách nhiệm của AI1, AI2A và AI2B | Tránh viết Tuần 8 chỉ như một trang thu thập raw traffic | Xây dựng attack profile coverage matrix |
| Review baseline normal và AI2A | 08/06/2026 | Review tóm tắt dataset normal P01–P12 và contract dataset classifier AI2A | Xác định đường cơ sở normal và chính sách schema AI2A | Giữ row count làm bằng chứng, hoãn diễn giải QA sâu hơn sang Tuần 9 | Review bằng chứng attack profile |
| Review coverage attack profile | 10/06/2026 | Kiểm tra folder, summary, merged dataset, QA/report artifact của attack profile A01–A10 | A01–A06 và A08–A10 có bằng chứng artifact-backed; A07 có trong aggregate dataset evidence; A11 không tìm thấy | Không khẳng định A11 hoặc cường điệu trạng thái profile-level của A07 | Review artifact AI2B/A12 |
| Review bằng chứng HTTP semantic | 12/06/2026 | Review raw HTTP log SQLi/XSS của A12 và artifact dataset/report của AI2B | A12 được xác nhận là nguồn HTTP semantic cho AI2B | Giữ A12 tách biệt khỏi coverage flow profile AI2A | Review context AI1 |
| Review AI1 và ranh giới claim | 13/06/2026 | Review handoff AI1, model card, feature manifest context và giới hạn đã biết | AI1 được ghi lại như anomaly-detection context thay vì training output Tuần 8 | Trích xuất feature live/replay từ backend vẫn là hạng mục tích hợp sau | Chuẩn bị docs Tuần 8 |
| Hợp nhất bằng chứng | 13/06/2026 | Xây dựng evidence index, source inventory, AI mapping và claim boundary Tuần 8 | Tuần 8 có thể được giải thích từ docs và bảng mà không cần screenshot placeholder | Ảnh chụp màn hình có thể được bổ sung sau nếu cần | Dùng Tuần 9 cho QA sâu hơn và xác minh model-readiness |

## Bố cục Workspace Dataset

| Khu vực workspace | Mục đích | Diễn giải Tuần 8 |
| --- | --- | --- |
| `Dataset/tools/final_dataset/` | Dataset normal P01–P12 và merge report | Artifact-backed |
| `Dataset/tools/attack_profiles/` | Attack campaign profile, profile summary, merged dataset và QA report | Nguồn chính cho attack coverage của AI2A |
| `Dataset/tools/attack_profiles/final_attack_dataset/` | Contract dataset classifier AI2A, merged CSV, QA report và training strategy | Anchor workspace dataset AI2A |
| `Dataset/tools/ai1_modeling/` | AI1 handoff, model card, feature manifest, threshold và smoke evidence | Bối cảnh anomaly-detection AI1 |
| `Dataset/tools/ai2a_modeling/` | Release-candidate AI2A và model-readiness report | Hoãn sang Tuần 9 để xác minh sâu hơn |
| `Dataset/tools/ai2b_modeling/` | HTTP semantic dataset AI2B, shortcut check, freeze review và holdout note | Hoãn sang Tuần 9 để xác minh sâu hơn |

## Đường cơ sở Dataset Bình thường

Đường cơ sở bình thường sử dụng tóm tắt dataset normal cuối cùng P01–P12 lưu trong `Dataset/tools/final_dataset/`. Các metric dưới đây là artifact-backed value từ existing summary report.

| Metric | Giá trị |
| --- | ---: |
| Rows | 197,797 |
| Columns | 70 |
| Source profile | 12 |
| Rows `missed_bytes` | 0 |
| Rows IPv6 | 0 |
| Duplicate UID | 0 |
| Missing cells | 0 |
| Dropped missing rows | 154 |
| Dropped duplicate UID | 0 |

Reviewed report mô tả một corpus tham chiếu bình thường lớn cho AI1 và AI2A. Tuần 8 ghi lại đây là bằng chứng workspace dataset. Xác minh sâu hơn về split suitability, leakage risk và model readiness được hoãn sang Tuần 9.

## Phủ sóng Attack Profile của AI2A

AI2A là classifier flow-level có giám sát. Nó sử dụng Zeek `conn.log`-derived feature và ánh xạ bằng chứng attack-profile vào các class có giám sát như `port_scan_or_recon`, `ssh_bruteforce_indicator` và `web_initial_access_indicator`.

Bằng chứng dataset classifier AI2A đến từ contract dataset classifier cuối cùng lưu trong `Dataset/tools/attack_profiles/final_attack_dataset/`. Các metric dưới đây là artifact-backed value.

| Metric | Giá trị |
| --- | ---: |
| Rows | 207,881 |
| Columns | 77 |
| Run count | 281 |
| Model feature count | 53 |
| Duplicate UID | 0 |

Tuần 8 dùng các con số này để chứng minh workspace dataset tồn tại và được tổ chức. Model-readiness cuối cùng, đánh giá release-candidate và chi tiết xác minh per-class được dành cho Tuần 9.

| Profile | AI2A label hoặc vai trò | Trạng thái Tuần 8 | Diễn giải bằng chứng |
| --- | --- | --- | --- |
| A01 Discovery | `port_scan_or_recon` | Artifact-backed | Tìm thấy final summary và merged dataset evidence |
| A02 SSH authentication | `ssh_bruteforce_indicator` | Artifact-backed | Tìm thấy core và holdout dataset report. SSH flow evidence tồn tại; vì SSH authentication content được mã hóa, Zeek `conn.log` đơn lẻ không cung cấp kết quả xác thực dứt khoát |
| A03 Web initial access | `web_initial_access_indicator` | Artifact-backed | Tìm thấy final summary và core/holdout dataset report |
| A04 HTTP beaconing | `http_beaconing_indicator` | Artifact-backed | Tìm thấy final summary và dataset evidence |
| A05 DNS anomaly | `dns_anomaly_indicator` | Artifact-backed | Tìm thấy final summary và dataset evidence |
| A06 Collection staging | `collection_staging_indicator` | Artifact-backed | Tìm thấy final summary và dataset evidence |
| A07 Controlled exfiltration | `controlled_exfiltration` | Aggregate-only | Được tham chiếu trong tài liệu aggregate AI2A dataset; không tìm thấy standalone profile-level final summary trong lần quét hiện tại |
| A08 Suspicious admin | `suspicious_admin` | Artifact-backed | Tìm thấy final summary và QA/report evidence |
| A09 East-west simulation | `east_west_simulation` | Artifact-backed | Tìm thấy setup summary, final summary và QA/report evidence |
| A10 Mixed attack chain | Per-flow stage label | Artifact-backed | Tìm thấy final summary và QA/report evidence |
| A11 | Không khẳng định | Not found | Không tìm thấy artifact hiện có, nên Tuần 8 không khẳng định A11 |

## Phủ sóng HTTP Semantic của AI2B

AI2B tách biệt với AI2A. Nó tập trung vào HTTP request semantic thay vì phân loại network-flow tổng quát.

Bằng chứng Tuần 8 chính đến từ A12 Web Semantic artifact:

| Khu vực bằng chứng | Diễn giải |
| --- | --- |
| A12 SQLi HTTP log | Zeek `http.log` row chứa normalized URI/query evidence kiểu SQLi |
| A12 XSS HTTP log | Zeek `http.log` row chứa normalized URI/query evidence kiểu XSS |
| AI2B HTTP dataset | Structured HTTP semantic dataset được xây dựng cho `NONE`, `SQLI` và `XSS` |
| AI2B freeze/holdout report | Bối cảnh model-readiness hữu ích, nhưng thảo luận sâu hơn được hoãn sang Tuần 9 |

Tuần 8 có thể nêu an toàn rằng A12 là nguồn HTTP semantic evidence cho AI2B. Tuần 8 không khẳng định final AI2B blind-holdout metric.

AI2B tập trung vào normalized URI/query và request-line evidence. Tuần 8 không khẳng định HTTP request-body inspection, full payload coverage, cookie/session inspection hay phân tích browser-rendered content.

## Bối cảnh Dataset Anomaly của AI1

AI1 là component phát hiện anomaly. Handoff và model card được review mô tả một Isolation Forest model kỳ vọng một 30-feature vector được trích xuất từ Zeek `conn.log` row.

Tuần 8 xử lý AI1 như dataset-architecture context:

| Hạng mục | Ghi chú Tuần 8 |
| --- | --- |
| Mục đích model | Phát hiện network anomaly |
| Input kỳ vọng | Flow feature vector từ Zeek `conn.log` |
| Feature count | 30 |
| Loại quyết định | `NORMAL` hoặc `ANOMALY` |
| Giới hạn đã biết | Backend chưa trích xuất trực tiếp AI1 feature từ raw `conn.log` row |

Bằng chứng triển khai và API integration cho AI1 thuộc về công việc backend và model-readiness sau, không phải dataset coverage Tuần 8.

## Ma trận Phủ sóng Bằng chứng

| Khu vực | Bằng chứng chính | Trạng thái | Claim Tuần 8 |
| --- | --- | --- | --- |
| Normal P01–P12 | Final normal dataset summary (`Dataset/tools/final_dataset/`) | Artifact-backed | Đường cơ sở traffic bình thường tồn tại và có quality metric |
| AI2A profile coverage | Attack profile summary (`Dataset/tools/attack_profiles/`), final classifier dataset | Artifact-backed / aggregate-only theo profile | AI2A có workspace dataset flow-classification có cấu trúc |
| A02 SSH | Core/holdout dataset report (`Dataset/tools/attack_profiles/`) | Artifact-backed with limitation | SSH flow evidence tồn tại; encrypted auth result không thể quan sát trong Zeek `conn.log` |
| A07 exfiltration | Final AI2A aggregate dataset và strategy evidence | Aggregate-only | Được tham chiếu trong aggregate AI2A dataset; không khẳng định profile-level final summary |
| A11 | Không tìm thấy artifact | Not found | Loại trừ khỏi claim Tuần 8 |
| A12 HTTP semantic | Raw HTTP log và AI2B artifact (`Dataset/tools/ai2b_modeling/`) | Artifact-backed | A12 hỗ trợ AI2B normalized URI/query semantic evidence |
| AI1 | Handoff, model card, feature manifest context | Context-only | Yêu cầu input dataset AI1 đã được hiểu |

## Ranh giới Khẳng định và Khoảng thiếu

Claim an toàn của Tuần 8:

- Một workspace dataset được tổ chức cho AI1, AI2A và AI2B.
- Traffic bình thường P01–P12 tồn tại như một validated baseline dataset.
- AI2A sử dụng Zeek `conn.log` flow-level feature và attack profile A01–A10.
- AI2B sử dụng HTTP semantic evidence từ công việc A12 SQLi/XSS.
- A02 có validated dataset report, với giới hạn SSH encryption được giải thích rõ.
- A11 không được khẳng định vì không tìm thấy bằng chứng.

Claim ngoài phạm vi:

- Toàn bộ dải attack profile đánh số đã hoàn chỉnh.
- Dataset đã sẵn sàng cho sử dụng production.
- Hoàn thành training model thuộc về công việc xác minh sau.
- Split policy cấp production đã hoàn chỉnh.
- AI2B blind-holdout score có sẵn.
- A07 có profile-level final summary.

## Vấn đề và Quyết định

| Vấn đề / quan sát | Quyết định |
| --- | --- |
| Workspace dataset chứa artifact cho nhiều AI component | Tách biệt ownership theo AI1, AI2A và AI2B |
| A07 thiếu standalone final summary | Báo cáo là aggregate-only |
| Không tìm thấy artifact A11 | Không khẳng định A11 |
| SSH authentication content được mã hóa | Giới hạn claim A02 chỉ ở flow-pattern indicator |
| AI2B sử dụng normalized URI/query/request-line evidence | Không khẳng định request-body inspection hoặc full payload coverage |
| Các metric hiện có đến từ saved report, không phải re-run script | Gắn nhãn chúng là artifact-backed |
| Trích xuất AI1 live feature chưa hoàn chỉnh | Hoãn tích hợp backend sang các tuần sau |
| Model-readiness report rất rộng | Chuyển đánh giá sâu hơn sang Tuần 9 |

## Bài học rút ra

Bài học chính từ Tuần 8 là một security dataset không nên chỉ được mô tả bởi số lượng row đã thu thập. Câu hỏi quan trọng hơn là liệu mỗi dataset artifact có nguồn rõ ràng, AI owner rõ ràng, ý nghĩa nhãn có thể bảo vệ được và giới hạn đã biết hay không.

Một bài học khác là các AI component khác nhau cần bằng chứng khác nhau. AI2A có thể chủ yếu dựa vào Zeek `conn.log` flow-level feature, trong khi AI2B cần HTTP request-level evidence. AI1 có một anomaly-detection feature contract riêng biệt và không nên được trộn vào supervised classifier dataset nếu không có ranh giới rõ ràng.

## Kết quả đạt được trong tuần

Tuần 8 tạo ra một bản đồ rõ ràng về AI dataset workspace. Dự án hiện có mối quan hệ được ghi lại giữa IDS telemetry cục bộ, normal traffic profile, attack profile và dataset riêng cho từng AI component.

Worklog Tuần 8 cố ý dừng ở lớp coverage và tổ chức. Tuần 9 sẽ đi sâu hơn vào dataset QA, leakage/split policy, bằng chứng model-readiness AI2A và giới hạn freeze/holdout AI2B.

## Kế hoạch tuần tiếp theo

Trong Tuần 9, worklog sẽ tập trung vào AI dataset QA và xác minh model-readiness:

- AI1 anomaly model handoff và feature contract.
- AI2A classifier dataset QA, phân phối class, split policy và release-candidate evidence.
- AI2B HTTP semantic dataset QA, shortcut check, freeze review và ranh giới holdout chưa hoàn chỉnh.
- Giải thích rõ ràng những gì sẵn sàng cho demo và những gì vẫn là công việc xác minh trong tương lai.
