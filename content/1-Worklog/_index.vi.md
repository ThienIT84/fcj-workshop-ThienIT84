---
title: "Nhật ký công việc"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

Nhật ký này ghi lại **12 tuần thực tập** tại **Công ty TNHH Amazon Web Services Việt Nam** (20/04/2026 – 12/07/2026), thực hiện đề tài **AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform** trong khuôn khổ chương trình First Cloud Journey (FCAJ).

Chương trình triển khai qua năm giai đoạn: xây dựng nền tảng AWS cloud, thiết lập lớp dữ liệu và phân phối tĩnh, dựng local IDS lab và thu thập dữ liệu huấn luyện AI, kiểm tra chất lượng dataset và tích hợp backend, và cuối cùng triển khai pipeline xử lý alert bất đồng bộ trên cloud trước khi lưu trữ evidence và publish báo cáo workshop.

## Tổng quan tiến trình

| Giai đoạn | Tuần | Nội dung chính |
| --- | --- | --- |
| 🏗️ Nền tảng AWS Cloud | 1 – 4 | IAM, VPC, EC2/Bastion, CloudWatch/Session Manager |
| 🗄️ Lớp dữ liệu & Phân phối | 5 – 6 | RDS PostgreSQL, S3/CloudFront |
| 🔬 Dựng local lab & Thu thập dữ liệu | 7 – 8 | pfSense/Kali/Zeek lab + traffic campaigns & tạo dataset |
| 🤖 Kiểm tra AI & Backend | 9 – 10 | QA model, FastAPI fusion layer |
| ☁️ Pipeline Cloud & Hoàn thiện | 11 – 12 | SQS/Worker/RDS pipeline, dọn dẹp, deploy Hugo/Vercel |

---

## Giai đoạn 1 — Nền tảng AWS Cloud

**[Tuần 1: Bảo mật tài khoản AWS & IAM Access Baseline](1.1-week1/)**
Bảo vệ root user, bật MFA, đặt billing alarm, chọn region Singapore. Thiết lập access baseline cho quản trị viên, lập kế hoạch truy cập nhóm và thiết kế nhóm/quyền theo nguyên tắc least-privilege. Giai đoạn lab thực hành IAM users và group-based policies; triển khai thực tế dự án sau đó dùng AWS IAM Identity Center với permission sets và account assignments để kiểm soát truy cập nhóm. Các nhóm role builder được định nghĩa cẩn thận và không cấp quyền deploy rộng trong giai đoạn này.

**[Tuần 2: Nền tảng mạng VPC & Phân vùng subnet](1.2-week2/)**
VPC tùy chỉnh `10.10.0.0/16` với 2 public và 2 private subnet trải dài 2 Availability Zone. Internet Gateway định tuyến cho public subnet, liên kết route table và thiết kế ranh giới Security Group. Compute và NAT Gateway được chuyển sang Tuần 3.

**[Tuần 3: EC2, Bastion Access & NAT Private Egress](1.3-week3/)**
Triển khai EC2 ở public và private subnet dựa trên VPC Tuần 2. SSH bastion từ laptop qua public instance vào private instance. Cấp phát Elastic IP, NAT Gateway cho outbound traffic của private subnet, quản lý key pair. Công cụ vận hành chuyển sang Tuần 4.

**[Tuần 4: Vận hành an toàn & Quan sát hệ thống](1.4-week4/)**
AWS Systems Manager Session Manager và EC2 Instance Connect Endpoint để truy cập private mà không cần mở cổng SSH. VPC Flow Logs gửi log vào Amazon CloudWatch Logs. Metric filter cho traffic bị từ chối và CloudWatch alarm được ghi lại ở trạng thái `ALARM` kèm minh chứng.

---

## Giai đoạn 2 — Lớp dữ liệu & Phân phối

**[Tuần 5: Lớp dữ liệu RDS PostgreSQL](1.5-week5/)**
Amazon RDS PostgreSQL private đặt trong DB Subnet Group trải 2 AZ. Security Group kiểm soát truy cập từ EC2 client trên cổng 5432. Tạo schema bảng `alerts` với các trường `severity`, `message`, và `anomaly_score`. Xác thực kết nối bằng `psql` theo luồng: Laptop → EC2 → RDS private.

**[Tuần 6: S3 Static Website, CloudFront & Object Lifecycle](1.6-week6/)**
S3 object storage private để lưu raw dataset dạng Zeek. S3 static content hosting cho các file public được chọn lọc. Amazon CloudFront phân phối được cấu hình để phân phối nội dung frontend tĩnh từ S3 origin. S3 Versioning bật để bảo vệ rollback. Lifecycle rule cấu hình dọn dẹp phiên bản cũ và delete marker.

---

## Giai đoạn 3 — Dựng local lab & Thu thập dữ liệu

**[Tuần 7: Dựng Local IDS Lab & Kiểm thử Zeek Telemetry](1.7-week7/)**
Dựng và xác thực local Intrusion Detection lab gồm:
- **pfSense** làm virtual firewall/router phân vùng mạng
- **Kali Linux** làm host tấn công ở segment DMZ/OPT1 (`10.10.10.0/24`)
- **Ubuntu Victim** làm host đích ở segment LAN (`192.168.1.0/24`)
- **Zeek sensor** giám sát traffic LAN từ cùng segment

Kiểm thử rule firewall (allow và default-deny), xác nhận Zeek thấy được packet bằng `tcpdump`, kiểm tra các trường `conn.log` và `http.log` với UID correlation giữa lớp mạng và HTTP. Xuất Zeek telemetry thành CSV có cấu trúc và kiểm tra đầu ra bằng Pandas. Kibana visualization chuyển sang sau.

**[Tuần 8: Chạy Traffic Campaign & Tổ chức Dataset AI](1.8-week8/)**
Chạy các chiến dịch traffic có kiểm soát qua local IDS lab để tạo dataset có nhãn:
- **Traffic normal:** 12 baseline profile (P01–P12) phủ hành vi mạng thông thường
- **Traffic tấn công:** 10+ attack profile (A01–A06, A08–A10 có artifact; A07 chỉ aggregate; A12 HTTP semantic)

Tổ chức workspace dataset cho ba thành phần AI:
- **AI1** — Isolation Forest anomaly detector dùng 30 feature từ Zeek `conn.log`
- **AI2A** — Flow classifier dùng dataset Zeek conn 207.881 dòng (77 cột, 53 model feature)
- **AI2B** — HTTP semantic classifier (SQLi/XSS) dùng URI/query chuẩn hóa từ Zeek `http.log`

A11 không tìm thấy trong workspace và không được claim. A07 chỉ có trong tài liệu dataset tổng hợp.

---

## Giai đoạn 4 — Kiểm tra AI & Backend

**[Tuần 9: QA Dataset AI, Mức sẵn sàng Model & Ranh giới Validation](1.9-week9/)**
Rà soát ma trận QA qua AI1, AI2A và AI2B. Kiểm tra phân phối lớp, chính sách tách split/holdout, đánh giá release-candidate AI2A, và ghi nhận ranh giới freeze và blind-holdout của AI2B. Tạo bảng tóm tắt sẵn sàng với ranh giới claim rõ ràng, phân biệt phần đã xác thực và phần cần làm tiếp.

**[Tuần 10: Bàn giao Model AI, Backend API & Fusion Layer](1.10-week10/)**
Backend FastAPI với endpoint tiếp nhận chính `POST /api/events`. Logic định tuyến AI: flow evidence → AI1/AI2A, HTTP evidence → AI2B. Multi-model fusion layer xuất ra `attack_type`, `risk_score`, `severity`, danh sách contributor và cờ model bị loại trừ. Hợp đồng model adapter được định nghĩa cho từng thành phần AI.

---

## Giai đoạn 5 — Pipeline Cloud & Hoàn thiện

**[Tuần 11: Pipeline Alert Async AWS: SQS, Worker, RDS & Idempotency](1.11-week11/)**
Triển khai pipeline xử lý alert bất đồng bộ: `POST /api/events/http/async` → SQS main queue → `socai-ai-worker` consumer → bảng RDS `final_alerts`. AWS Secrets Manager dùng để lấy credentials RDS. Kiểm thử D5-5 idempotency bằng cách gửi lại cùng một `event_id` ba lần; `final_alerts` chỉ giữ một dòng với payload mới nhất. `GET /api/alerts/latest` xác nhận lấy alert đã persist qua API.

**[Tuần 12: Demo cuối, Dọn dẹp AWS, Lưu trữ Evidence & Deploy Workshop](1.12-week12/)**
Minh chứng demo cuối xác nhận alert SQL Injection với mức nghiêm trọng critical, evidence AI2B, fusion confidence và MITRE mapping hiển thị qua API và dashboard đã triển khai. Hoàn thành checklist lưu trữ và masking evidence. Dọn dẹp AWS theo thứ tự dependency bao gồm compute, load balancing, queue, database, storage, secrets, CloudWatch, IAM và tài nguyên mạng. RDS bị xóa kèm final snapshot, việc xóa Secrets Manager được lên lịch với recovery window. CloudFront distribution, WAF Web ACL và OAC được ghi nhận là deferred do ràng buộc của gói CloudFront flat-rate pricing-plan. Workshop Hugo publish qua Vercel làm bài nộp cuối kỳ FCAJ.