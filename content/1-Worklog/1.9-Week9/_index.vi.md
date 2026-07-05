---
title: "Tuần 9 - Dataset QA AI, Model Readiness & Ranh giới Xác minh"
date: 2026-06-13
weight: 9
chapter: false
draft: true
pre: " <b> 1.9. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Trọng tâm chính | AI dataset QA và review model-readiness |
| Mục tiêu chính | Đánh giá mức độ sẵn sàng demo/tích hợp của AI1, AI2A và AI2B |
| Trạng thái AI1 | Reviewed artifact contract phát hiện anomaly |
| Trạng thái AI2A | Reviewed classifier dataset QA và release-candidate evidence |
| Trạng thái AI2B | Reviewed HTTP semantic dataset, freeze review và holdout boundary |
| Đầu ra chính | QA matrix, readiness summary và validation boundary |
| Trạng thái trong tuần | Triển khai dạng docs-first draft; ảnh chụp màn hình và time log chính xác cần chủ nhân xác nhận |

## Lưu ý về Nguồn Bằng chứng

Tuần 9 sử dụng bằng chứng dataset và modeling được chuẩn bị trong project dataset workspace. Worklog công khai chỉ tham chiếu đường dẫn dự án logic:

```text
Dataset/tools/ai1_modeling/
Dataset/tools/attack_profiles/final_attack_dataset/
Dataset/tools/ai2a_modeling/
Dataset/tools/ai2b_modeling/
```

Ghi chú nội bộ Tuần 9 được lưu trong:

```text
docs/worklog/week9/
```

Không có raw model file, `.joblib` file, CSV, JSON report, local machine path hay private environment file nào được công khai trên trang Hugo này.

## Mục tiêu

Mục tiêu của Tuần 9 là đánh giá bằng chứng AI dataset và model-readiness sau khi workspace dataset đã được tổ chức trong Tuần 8.

Tuần 9 trả lời bốn câu hỏi:

- AI1 có artifact contract phát hiện anomaly rõ ràng không?
- AI2A có classifier dataset được xác minh và QA report không?
- AI2A có release-candidate evidence có thể giải thích một cách trung thực không?
- AI2B có HTTP semantic QA evidence và holdout boundary rõ ràng không?

## Bối cảnh Dự án

Tuần 8 đã ánh xạ workspace dataset và profile coverage. Tuần 9 đi sâu hơn một cấp vào chất lượng và mức độ sẵn sàng.

Mục tiêu không phải là khẳng định mọi model đều sẵn sàng cho hoạt động dài hạn. Mục tiêu là ghi lại phần nào sẵn sàng cho demo hoặc tích hợp, phần nào là release candidate và phần nào vẫn cần xác minh trong tương lai.

Đường review là:

```text
AI1 artifact contract
-> AI2A classifier dataset QA
-> AI2A release-candidate report
-> AI2B HTTP dataset và shortcut check
-> AI2B freeze review và holdout boundary
-> final readiness matrix
```

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày | Thời lượng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- | --- |
| Review mức độ sẵn sàng AI1 | 13/06/2026 | Ước tính 3.5 giờ | Review AI1 handoff, model card, feature list, threshold policy, smoke artifact và giới hạn đã biết | AI1 có artifact contract phát hiện anomaly được ghi lại | Backend vẫn cần raw `conn.log` để AI1 feature extraction cho live/replay | Review AI2A dataset QA |
| Review AI2A dataset QA | 14/06/2026 | Ước tính 5 giờ | Review classifier dataset report AI2A, QA report, class count, source row, run count, model feature và metadata bị loại trừ | AI2A dataset có 207,881 rows, 77 columns, 281 run, 53 model feature và duplicate UID 0 | Chất lượng dataset có thể giải thích được, nhưng giới hạn rare-class phải vẫn hiển thị | Review AI2A release candidate |
| Review AI2A release candidate | 15/06/2026 | Ước tính 5 giờ | Review selected candidate, validation metric, threshold freeze, release gate check, per-class metric và no-collapse watch table | AI2A release candidate được chấp nhận với validation macro F1 0.835883 và weighted F1 0.985125 | Holdout và rare-class drop cần diễn giải cẩn thận | Review AI2B HTTP semantic evidence |
| Review dataset và baseline AI2B | 16/06/2026 | Ước tính 5 giờ | Review AI2B HTTP semantic dataset summary, baseline report, shortcut sanity, endpoint-only baseline, parameter-only baseline và label-shuffle sanity | AI2B HTTP semantic dataset và baseline evidence có sẵn cho `NONE`, `SQLI` và `XSS` | Strong validation metric phải được trình bày với shortcut và holdout discipline | Review AI2B freeze/holdout boundary |
| AI2B freeze và holdout boundary | 17/06/2026 | Ước tính 4.5 giờ | Review V1.4.9 freeze review và incomplete holdout summary | AI2B candidate được freeze, nhưng holdout attempt không được tính điểm vì protocol guard phát hiện contamination hoặc inventory mismatch trước khi prediction | Trình bày đây là validation discipline, không phải model prediction failure | Hợp nhất readiness matrix |
| Hợp nhất bằng chứng | 18/06/2026 | Ước tính 3.5 giờ | Xây dựng QA summary nội bộ Tuần 9, screenshot checklist và validation boundary note | Tuần 9 sẵn sàng cho trang docs-first draft | Có thể thêm ảnh chụp màn hình công khai sau nếu cần | Chuyển Tuần 10 sang thiết kế API/backend/fusion |

## Tổng quan QA Dataset AI

| Component | Mức độ sẵn sàng | Bằng chứng reviewed | Ranh giới chính |
| --- | --- | --- | --- |
| AI1 | Artifact contract sẵn sàng cho integration planning | Handoff, model card, feature manifest, threshold, smoke response | Trích xuất live/replay từ raw `conn.log` vẫn cần triển khai |
| AI2A | Dataset QA và release-candidate evidence có sẵn | Classifier dataset report, QA report, release-candidate report | Hiệu năng rare-class và diễn giải holdout phải vẫn hiển thị |
| AI2B | HTTP semantic candidate frozen với strict holdout boundary | HTTP dataset summary, baseline report, freeze review, incomplete holdout summary | Holdout attempt bị chặn trước khi tính điểm bởi protocol guard |

## Mức độ Sẵn sàng Anomaly Model AI1

AI1 là component phát hiện network anomaly. Bằng chứng reviewed cho thấy AI1 có artifact contract có thể được tích hợp bởi backend khi feature extraction sẵn sàng.

| Hạng mục | Bằng chứng |
| --- | --- |
| Loại model | Isolation Forest |
| Mục đích | Phát hiện network anomaly từ Zeek `conn.log` flow feature |
| Feature cần thiết | 30-feature vector |
| Hướng điểm | Confidence cao hơn nghĩa là anomalous hơn |
| Threshold | `0.398066` |
| Đầu ra quyết định | `NORMAL` hoặc `ANOMALY` |
| Smoke evidence | API smoke response tồn tại trong artifact folder |

Giới hạn quan trọng:

```text
AI1 does not consume raw conn.log rows directly in the current backend path.
A feature extraction step is still required before live/replay traffic can be
sent to the AI1 adapter.
```

## QA Dataset Classifier AI2A

AI2A là supervised flow-level classifier. Nó tiêu thụ Zeek `conn.log`-derived feature và dự đoán một network attack class.

Classifier dataset QA evidence báo cáo:

| Metric | Giá trị |
| --- | ---: |
| Rows | 207,881 |
| Columns | 77 |
| Run count | 281 |
| Target column | `ai2a_label` |
| Model feature count | 53 |
| Duplicate UID | 0 |
| Normal rows | 199,326 |
| Attack rows | 8,555 |

Class coverage bao gồm:

| Class | Rows |
| --- | ---: |
| `normal` | 199,326 |
| `web_initial_access_indicator` | 3,811 |
| `ssh_bruteforce_indicator` | 2,177 |
| `port_scan_or_recon` | 1,035 |
| `dns_anomaly_indicator` | 617 |
| `http_beaconing_indicator` | 498 |
| `collection_staging_indicator` | 212 |
| `controlled_exfiltration` | 70 |
| `suspicious_admin` | 69 |
| `east_west_simulation` | 66 |

Kết quả QA hữu ích vì nó tách biệt model feature khỏi metadata. Các trường như `uid`, `run_id`, timestamp, IP identity, event metadata, provenance label và target label bị loại trừ khỏi model feature.

## Xác minh Release Candidate AI2A

AI2A release report chọn:

```text
rf_v2_1_full_safe_plus_ssh_minimal
```

Kết quả checkpoint:

```text
RELEASE_CANDIDATE_ACCEPTED
```

Validation metric cho candidate được chọn:

| Metric | Giá trị |
| --- | ---: |
| Validation macro F1 | 0.835883 |
| Validation weighted F1 | 0.985125 |
| Validation normal FPR | 0.003624 |
| A02 recall | 0.985455 |
| A03 recall | 0.861905 |
| A07 recall | 0.888889 |
| A08 recall | 0.636364 |
| A09 recall | 0.636364 |

Release-candidate evidence đủ mạnh cho thảo luận demo/model-readiness. Vẫn quan trọng để hiển thị hành vi cấp class vì một số rare class có recall thấp hơn hoặc fragile hơn so với dominant normal class.

## QA Dataset HTTP Semantic AI2B

AI2B là HTTP semantic detector cho:

```text
NONE
SQLI
XSS
```

Bằng chứng HTTP dataset V1.4.8J bao gồm:

| Hạng mục | Giá trị |
| --- | --- |
| Dataset rows | 11,829 |
| Duplicate request key | 0 |
| Diagnostic overlap audit | Passed |
| Nguồn chính | A12 Web Semantic HTTP evidence |

Baseline report chọn:

```text
tfidf_char_lex_v4_logreg_uri_query_norm_no_ident_no_class_weight
```

Shortcut sanity check quan trọng vì một HTTP semantic detector có thể vô tình học endpoint hoặc parameter shortcut thay vì payload semantic.

| Kiểm tra | Kết quả |
| --- | --- |
| Endpoint-only baseline | Passed |
| Parameter-only baseline | Passed |
| Label-shuffle sanity | Passed |
| Metric stability | Passed |

## Freeze Review và Ranh giới Holdout AI2B

V1.4.9 freeze review kết luận:

```text
AI2B_CANDIDATE_FROZEN_FOR_HOLDOUT_121_124
```

Candidate:

```text
tfidf_char_lex_v4_logreg_uri_query_norm_no_ident_no_class_weight
```

Bằng chứng freeze:

| Hạng mục | Kết quả |
| --- | --- |
| Source lock | Passed |
| Canary | Passed |
| Candidate consistency | Passed |
| Threshold | `0.50` |

Holdout review được báo cáo một cách có chủ ý là incomplete. Protocol nghiêm ngặt đã chặn việc tính điểm trước khi prediction vì các holdout attempt có vấn đề contamination hoặc inventory mismatch:

| Attempt | Kết quả | Được tính điểm | Lý do |
| --- | --- | --- | --- |
| `121-124` | Contamination guard failed | Không | Exact overlap với release dataset |
| `125-128` | Inventory guard failed | Không | Actual row không có trong frozen planned inventory |
| `129-132` | Inventory guard failed | Không | Actual multiset khác với frozen manifest |

Đây là giới hạn của validation pipeline, không phải bằng chứng model dự đoán sai. Vì việc tính điểm không bắt đầu, Tuần 9 không báo cáo final holdout recall hoặc false-positive rate cho AI2B.

## Ma trận Xác minh

| Khu vực | Bằng chứng | Trạng thái | Diễn giải Tuần 9 |
| --- | --- | --- | --- |
| AI1 artifact contract | Handoff, model card, threshold, smoke response | Sẵn sàng cho integration planning | Vẫn cần feature extraction |
| AI2A dataset QA | Classifier report và QA report | Vượt qua core QA check | Cơ sở tốt cho thảo luận model-readiness |
| AI2A release candidate | Release report | Được chấp nhận | Strong demo candidate với class-level caveat |
| AI2B HTTP dataset | Dataset summary và baseline report | Baseline và shortcut check có sẵn | Semantic detector evidence mạnh nhưng vẫn có giới hạn |
| AI2B freeze review | V1.4.9 freeze review | Candidate frozen | Holdout evaluation vẫn chưa hoàn chỉnh |
| AI2B holdout attempt | Incomplete holdout summary | Không được tính điểm | Guardrail chặn tính điểm trước khi prediction |

## Quyết định Leakage, Split và An toàn

Quyết định an toàn quan trọng là tránh training trên identity hoặc provenance shortcut.

Đối với AI2A, các trường bị loại trừ bao gồm:

```text
uid
run_id
ts
flow_end_ts
src_ip
dst_ip
src_port
event metadata
source_profile
attack metadata
label
ai2a_label
```

Đối với AI2B, các report nhấn mạnh validation discipline:

- threshold selection chỉ sử dụng validation,
- holdout adjustment bị vô hiệu hóa,
- endpoint-only và parameter-only shortcut check được review,
- holdout attempt bị contaminated hoặc inventory mismatch không được tính điểm.

## Vấn đề và Giới hạn

| Giới hạn | Tác động | Hướng tiếp theo |
| --- | --- | --- |
| AI1 raw `conn.log` feature extraction chưa được tích hợp | Live/replay AI1 cần một extraction layer | Triển khai extraction trước khi khẳng định AI1 processing trực tiếp |
| AI2A rare class có support nhỏ hơn | Một số class metric fragile hơn | Giữ per-class metric hiển thị |
| AI2B holdout attempt không được tính điểm | Không thể báo cáo final AI2B holdout score | Sửa capture/inventory protocol trước lần one-shot attempt tiếp theo |
| Ảnh chụp màn hình chưa được đính kèm | Tuần 9 hiện là docs-first | Thêm selected redacted screenshot sau nếu cần |

## Bài học rút ra

Tuần 9 cho thấy model readiness không chỉ là validation score cao. Một AI security report hữu ích cũng phải giải thích feature contract, leakage boundary, split discipline, class imbalance, shortcut check và holdout status.

Bài học quan trọng nhất đến từ AI2B: một strict validation protocol có thể chặn việc tính điểm trước khi prediction. Điều đó không thoải mái, nhưng trung thực hơn là tính điểm trên dữ liệu holdout bị contaminated hoặc lệch lạc.

## Kết quả đạt được trong tuần

Tuần 9 tạo ra một cái nhìn readiness rõ ràng cho AI1, AI2A và AI2B.

AI1 có artifact contract và smoke evidence. AI2A có classifier dataset QA và một accepted release candidate. AI2B có HTTP semantic dataset evidence và một frozen candidate, nhưng strict holdout evaluation của nó vẫn là công việc xác minh trong tương lai.

## Kế hoạch tuần tiếp theo

Tuần 10 sẽ chuyển từ bằng chứng dataset/model sang thiết kế hệ thống và tích hợp:

- định nghĩa API contract cho AI1, AI2A, AI2B và fusion,
- mô tả backend request/response flow,
- ghi lại cách alert evidence trở thành risk score,
- chuẩn bị đường dẫn cho AWS MVP migration và SOC demo.
