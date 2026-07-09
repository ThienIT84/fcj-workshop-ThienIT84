---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform
## Nền tảng SOC Hybrid Cloud tích hợp AI đa mô hình cho giám sát và phản ứng bảo mật

### 1. Tóm tắt điều hành
AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform là nền tảng giám sát bảo mật hybrid cloud được đề xuất nhằm kết nối Local Security Lab với hạ tầng AWS để thu thập log, xử lý sự kiện, chạy AI inference và hiển thị cảnh báo realtime trên SOC Dashboard. Hệ thống sẽ kết hợp dữ liệu từ pfSense, Zeek, Suricata và collector/tailer trong lab với các dịch vụ AWS như CloudFront, WAF, ALB, EC2, SQS, RDS PostgreSQL, S3, CloudWatch, CloudTrail, SNS, IAM, Secrets Manager và KMS.

Mục tiêu cuối cùng của dự án là xây dựng một kiến trúc production-style có khả năng tiếp nhận sự kiện bảo mật qua HTTPS, chuẩn hóa dữ liệu ở backend ingestion API, đẩy message vào SQS để xử lý bất đồng bộ, chạy các mô hình AI chuyên biệt và tổng hợp kết quả bằng Fusion Layer. Kết quả cuối cùng sẽ được lưu vào RDS/S3 và hiển thị trên dashboard với severity, risk score, confidence score, MITRE ATT&CK mapping, evidence và decision flow.

Nền tảng sử dụng ba hướng phân tích AI chính: AI1 cho phát hiện bất thường trên network flow, AI2A cho phân loại network attack từ Zeek `conn.log`, và AI2B cho phát hiện HTTP semantic web attack như SQL Injection hoặc Cross-Site Scripting từ Zeek `http.log`. Fusion Layer sẽ tổng hợp kết quả AI và rule-based evidence để tạo final alert phục vụ điều tra và phản ứng sự cố.

### 2. Tuyên bố vấn đề
*Vấn đề hiện tại*

Trong các môi trường lab bảo mật hoặc SOC quy mô nhỏ, dữ liệu thường bị phân tán giữa nhiều nguồn như Zeek `conn.log`, Zeek `http.log`, Suricata alerts, pfSense logs và ghi chú pentest thủ công. Việc đọc log thủ công làm chậm quá trình phát hiện sự cố, khó liên kết bằng chứng theo thời gian và không phù hợp khi số lượng event tăng lên trong các attack campaign.

Các hệ thống rule-based truyền thống có thể phát hiện một số dấu hiệu rõ ràng, nhưng dễ bỏ sót hành vi bất thường hoặc web attack được encode/obfuscate. Ngược lại, nếu chỉ dùng một mô hình AI đơn lẻ, hệ thống thiếu ngữ cảnh, khó giải thích quyết định và dễ tạo false positive. Một SOC dashboard hiệu quả cần kết hợp nhiều nguồn evidence, nhiều mô hình AI và một lớp fusion có khả năng đưa ra cảnh báo cuối cùng có thể giải thích được.

*Giải pháp*

Dự án đề xuất một kiến trúc hybrid cloud trong đó Local Collector không gửi trực tiếp vào SQS. Thay vào đó, collector sẽ gửi event qua HTTPS tới CloudFront/WAF, đi qua ALB và backend ingestion API. Backend sẽ xác thực, chuẩn hóa schema, kiểm tra dữ liệu đầu vào và đẩy message vào SQS. Worker service trong private subnet sẽ consume message từ SQS, chạy AI inference, thực hiện Fusion và ghi final alert vào RDS PostgreSQL/S3.

Dashboard React/Vite sẽ được build thành static files và lưu trong private S3 frontend bucket. CloudFront sẽ dùng Origin Access Control để đọc object từ S3 và phục vụ dashboard qua HTTPS. Backend API và WebSocket service sẽ được expose qua ALB, trong khi worker tier sẽ không public và chỉ giao tiếp với SQS, RDS, S3, Secrets Manager và CloudWatch qua quyền IAM phù hợp.

*Lợi ích và hoàn vốn đầu tư (ROI)*

Giải pháp giúp nhóm xây dựng một nền tảng SOC có khả năng demo end-to-end từ lab attack đến final alert trên dashboard. Hệ thống giảm thời gian đọc log thủ công, tạo evidence có cấu trúc, hỗ trợ analyst hiểu vì sao một alert được tạo và cung cấp nền tảng để tiếp tục cải tiến AI models. Về mặt kiến trúc, cách tách backend tier và worker tier giúp nhóm kiểm soát tốt hơn giữa realtime API/WebSocket workload và AI processing workload, đồng thời tạo nền tảng để scale theo request, CPU hoặc queue backlog.

### 3. Kiến trúc giải pháp
Proposal này mô tả kiến trúc mục tiêu cuối cùng của dự án. Việc triển khai sẽ được chia theo các phase để giảm rủi ro, kiểm thử từng lớp và kiểm soát chi phí.

#### 3.1. Sơ đồ kiến trúc mục tiêu

![Hybrid SOC Target Architecture](/images/2-Proposal/hybrid_soc_target_architecture.jpg)

*Hình 1. Kiến trúc mục tiêu của nền tảng SOC hybrid cloud. Sơ đồ mô tả luồng từ Local Lab, CloudFront/WAF, ALB, backend private compute, SQS worker, AI/Fusion, RDS/S3 đến monitoring và governance.*

Luồng xử lý mục tiêu:

```text
Local Collector
  -> CloudFront / AWS WAF
  -> Application Load Balancer
  -> Backend ingestion API
  -> Amazon SQS
  -> Worker service
  -> AI1 / AI2A / AI2B + Fusion Layer
  -> Amazon RDS for PostgreSQL / Amazon S3
  -> Dashboard API + WebSocket
  -> SOC Dashboard
```

#### 3.2. Local Lab và nguồn log

Local Security Lab sẽ gồm attacker machine, pfSense firewall, victim machine, Zeek sensor và Zeek Collector/Tailer. Attack traffic sẽ đi qua pfSense và victim services, sau đó được Zeek quan sát để tạo `conn.log` và `http.log`. Suricata hoặc rule-based evidence có thể được bổ sung để tăng ngữ cảnh cho Fusion Layer.

Collector/Tailer sẽ chịu trách nhiệm đọc log, chuẩn hóa các trường cần thiết, gắn timestamp/correlation id và gửi event qua HTTPS tới endpoint ingestion trên AWS. Collector không cần AWS credentials để ghi trực tiếp vào SQS, giúp giảm rủi ro lộ credential ở môi trường lab.

#### 3.3. Edge, frontend và CloudFront/WAF

Người dùng SOC sẽ truy cập dashboard qua CloudFront bằng HTTPS. Frontend React/Vite sẽ được build thành static files và lưu trong private S3 frontend bucket. Frontend bucket sẽ không public; CloudFront sẽ dùng Origin Access Control để đọc object từ S3 và phục vụ dashboard an toàn hơn.

AWS WAF sẽ được đặt ở lớp public entry để áp dụng Web ACL và managed rules cho các request đi vào. CloudFront/WAF giúp giảm rủi ro ở lớp edge, đồng thời tạo một điểm truy cập nhất quán cho dashboard, API và WebSocket.

Để tránh bypass CloudFront/WAF, ALB sẽ được cấu hình chỉ forward request hợp lệ khi có origin verification header do CloudFront thêm vào. Security Group của ALB cũng sẽ được giới hạn theo nguồn phù hợp, ví dụ CloudFront origin-facing prefix list trong giai đoạn triển khai. Cơ chế này giúp đảm bảo API/backend không bị truy cập trực tiếp từ Internet ngoài đường CloudFront đã kiểm soát.

#### 3.4. Backend API và WebSocket

Backend tier sẽ chạy FastAPI API service và WebSocket service trong private subnets. Backend được expose thông qua Application Load Balancer, không mở trực tiếp từ Internet tới EC2. Backend Auto Scaling Group có thể scale service theo request rate, CPU hoặc các metric phù hợp với API/WebSocket traffic.

Backend ingestion API sẽ thực hiện xác thực, schema validation, normalization và routing ban đầu. Sau khi event hợp lệ, backend sẽ gửi message vào SQS để worker xử lý bất đồng bộ. WebSocket service sẽ phục vụ realtime alert feed cho dashboard, trong khi REST API cung cấp alert list, alert detail, dashboard summary, model status và data source health.

#### 3.5. SQS và worker xử lý bất đồng bộ

Amazon SQS sẽ đóng vai trò queue nội bộ giữa backend ingestion API và worker tier. Queue giúp hấp thụ burst traffic từ lab, hỗ trợ retry và giảm coupling giữa request ingestion với AI inference. Dead-letter queue sẽ được dùng để giữ lại các message lỗi sau số lần retry nhất định.

Worker tier sẽ gồm SQS consumer và AI/Fusion processing worker, chạy trong private subnets và không expose public. Worker Auto Scaling Group có thể scale theo queue backlog hoặc message age. Worker sẽ consume message từ SQS, gọi AI adapters, chạy Fusion Layer, ghi final alert vào RDS/S3 và phát sinh telemetry phục vụ monitoring.

#### 3.6. AI/Fusion layer

AI layer sẽ gồm ba nhóm mô hình:

- *AI1 - Anomaly Detection*: phát hiện network flow bất thường từ Zeek `conn.log`.
- *AI2A - Network Attack Classification*: phân loại các hành vi như port scan, brute force, beaconing, web initial access hoặc network anomaly từ flow-level features.
- *AI2B - HTTP Semantic Web Attack Detection*: phát hiện SQL Injection, Cross-Site Scripting và các web attack semantic từ HTTP method, URI, query string hoặc payload metadata.

Fusion Layer sẽ tổng hợp output của các mô hình và rule-based evidence để tạo final alert. Alert sẽ có attack type, severity, risk score, confidence score, detected_by, MITRE ATT&CK mapping, raw/evidence summary và decision flow. Cách này giúp dashboard không chỉ hiển thị nhãn AI thô, mà hiển thị kết luận cuối cùng có thể giải thích được.

#### 3.7. RDS/S3 storage layer

Amazon RDS for PostgreSQL Multi-AZ deployment sẽ cung cấp primary DB instance và standby DB instance ở Availability Zone khác để tăng high availability và hỗ trợ failover. RDS PostgreSQL Multi-AZ sẽ lưu final alerts, evidence summaries và dữ liệu truy vấn dashboard. Standby instance được dùng cho high availability và failover, không phải là read-serving replica trong phạm vi thiết kế mặc định.

Amazon S3 Data Bucket sẽ lưu raw logs, processed evidence, model artifacts hoặc export reports tùy từng phase triển khai. S3 Gateway Endpoint sẽ được dùng cho backend/worker trong private subnets khi truy cập S3 Data Bucket, model artifacts hoặc evidence export mà không cần đi qua NAT Gateway. Đường truy cập frontend bucket vẫn do CloudFront thực hiện thông qua Origin Access Control.

#### 3.8. Monitoring, governance và incident response

CloudWatch sẽ thu thập logs, metrics và alarms cho backend, worker, queue, RDS và các thành phần vận hành. CloudTrail sẽ ghi nhận AWS API activity để phục vụ audit. SNS sẽ gửi email/SMS notification khi có alarm hoặc sự kiện vận hành quan trọng.

IAM sẽ được thiết kế theo least privilege cho backend, worker và deployment roles. Secrets Manager sẽ lưu DB credential/API secrets thay vì hard-code trong source code. KMS sẽ hỗ trợ mã hóa secrets và dữ liệu nhạy cảm. Security Groups sẽ giới hạn đường đi mạng giữa ALB, backend tier, worker tier, RDS và các dịch vụ liên quan.

#### 3.9. Quyết định kiến trúc và trade-off

| Quyết định | Lựa chọn | Lý do | Trade-off |
|---|---|---|---|
| Edge entry | CloudFront + WAF | Phân phối HTTPS, bảo vệ public entry và thống nhất đường vào dashboard/API | Cần cấu hình cache behavior và origin routing cẩn thận |
| Frontend hosting | Private S3 bucket + CloudFront OAC | Giữ bucket không public, CloudFront đọc object qua OAC | Phải cấu hình bucket policy đúng |
| API entry | ALB -> private backend EC2 | Tách public entry khỏi compute private, hỗ trợ health check và scale backend | ALB phát sinh chi phí theo giờ/LCU |
| Ingestion decoupling | SQS Standard + DLQ | Hấp thụ burst traffic, retry message lỗi, tách ingestion khỏi AI processing | SQS Standard có thể giao message nhiều hơn một lần, cần idempotency |
| AI processing | Worker tier riêng | Scale theo queue backlog và không chặn request API | Cần quản lý worker lifecycle và DLQ recovery |
| Final alert storage | RDS PostgreSQL | Phù hợp dashboard query, filter, alert history và evidence summary | Chi phí cao hơn lưu JSON đơn giản trên S3 |
| Raw/evidence storage | S3 Data Bucket | Phù hợp log, artifact, evidence export và chi phí thấp | Không tối ưu cho query transaction |
| Secret management | Secrets Manager + IAM role | Không hard-code DB/API secrets, hỗ trợ rotation | Cần kiểm soát IAM/KMS và không chụp secret value |

### 4. Triển khai kỹ thuật
*Các giai đoạn triển khai*

1. *Local Lab & log generation*: dựng attacker, pfSense, victim, Zeek sensor và collector/tailer; tạo traffic bình thường và attack campaign; thu `conn.log`, `http.log` và evidence.
2. *Frontend delivery with S3 and CloudFront*: xây dashboard React/Vite, build static assets, lưu vào private S3 bucket và phân phối qua CloudFront với Origin Access Control.
3. *Backend API, ALB and private compute*: triển khai FastAPI API/WebSocket service trong private subnets, expose qua ALB, cấu hình security groups và health checks.
4. *SQS, worker and asynchronous AI processing*: triển khai backend ingestion API đẩy event vào SQS, worker consume message, chạy AI/Fusion và xử lý retry/DLQ.
5. *RDS/S3 persistence and dashboard query layer*: triển khai RDS PostgreSQL, schema final alerts, S3 Data Bucket, API truy vấn dashboard và evidence/report export.
6. *Monitoring, governance, hardening and cost optimization*: triển khai CloudWatch, CloudTrail, SNS, IAM least privilege, Secrets Manager, KMS, AWS Budgets và cost-optimized operating mode.

*Yêu cầu kỹ thuật*

- *Backend/API*: FastAPI, WebSocket, REST APIs, schema validation, event normalization, SQS producer, dashboard query APIs.
- *Worker*: SQS consumer, batch processing, AI adapter orchestration, Fusion Layer, RDS/S3 writer, retry/DLQ handling.
- *Frontend*: React/Vite dashboard, realtime alert feed, KPI widgets, attack distribution charts, event detail modal, MITRE mapping và incident action simulation.
- *AI runtime*: AI1 anomaly detection, AI2A network attack classification, AI2B HTTP semantic web attack detection, confidence/risk scoring và model artifact management.
- *AWS infrastructure*: CloudFront, WAF, private S3 frontend bucket with OAC, ALB, EC2 Auto Scaling Groups, SQS/DLQ, RDS PostgreSQL Multi-AZ, S3 Data Bucket, CloudWatch, CloudTrail, SNS, IAM, Secrets Manager, KMS.

### 5. Lộ trình & Mốc triển khai
Proposal này mô tả kiến trúc mục tiêu cuối cùng của dự án. Việc triển khai sẽ được chia theo các phase để giảm rủi ro, kiểm thử từng lớp và kiểm soát chi phí.

- *Phase 1 - Local Lab & log generation*: dựng lab, tạo traffic, thu log và chuẩn hóa event contract.
- *Phase 2 - Frontend delivery with S3 and CloudFront*: triển khai dashboard static trên private S3 bucket, phân phối qua CloudFront/OAC.
- *Phase 3 - Backend API, ALB and private compute*: triển khai backend API/WebSocket service trong private subnets, expose qua ALB.
- *Phase 4 - SQS, worker and asynchronous AI processing*: thêm SQS queue/DLQ, worker tier, AI inference và Fusion xử lý bất đồng bộ.
- *Phase 5 - RDS/S3 persistence and dashboard query layer*: lưu final alerts/evidence vào RDS/S3, hoàn thiện dashboard APIs và report export.
- *Phase 6 - Monitoring, governance, hardening and cost optimization*: thêm CloudWatch, CloudTrail, SNS, IAM least privilege, Secrets Manager, KMS, AWS Budgets và tối ưu chi phí.

### 6. Ước tính ngân sách
Ước tính chi phí cho kiến trúc mục tiêu được xây dựng bằng [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=a306a7de1d836e86cd98fdd1980002edbde0cefb). Có thể tải [tệp ước tính chi phí](estimate_cost.pdf) để xem chi tiết.

*Target Architecture Cost Estimate*

Bản estimate hiện tại ghi nhận:

- *Upfront cost*: 0.00 USD.
- *Monthly cost*: 283.82 USD/tháng.
- *Total 12 months cost*: 3,405.84 USD.

Đây là ước tính cho target architecture, bao gồm các thành phần production-style như EC2, RDS PostgreSQL, VPC/NAT Gateway/public IPv4, Application Load Balancer, WAF, CloudWatch, Secrets Manager, KMS, S3 và SQS. Đây không phải là chi phí vận hành lab/demo tối ưu hằng ngày.

Các hạng mục chính theo estimate:

- *Amazon EC2*: 82.85 USD/tháng.
- *Amazon RDS for PostgreSQL*: 69.03 USD/tháng.
- *Amazon VPC/NAT/public IPv4*: 101.32 USD/tháng.
- *Elastic Load Balancing/Application Load Balancer*: 18.46 USD/tháng.
- *AWS WAF*: 8.03 USD/tháng.
- *Amazon CloudWatch*: 1.91 USD/tháng.
- *AWS Secrets Manager*: 0.85 USD/tháng.
- *AWS Key Management Service (KMS)*: 1.03 USD/tháng.
- *Amazon S3, Amazon SQS, Amazon CloudFront, Amazon SNS và AWS CloudTrail*: chi phí nhỏ hoặc gần 0 theo usage giả định trong estimate hiện tại.

Các số trong estimate phản ánh usage giả định tại thời điểm export. Những dịch vụ có chi phí theo request, data transfer hoặc log volume như S3, SQS, CloudFront, CloudWatch và WAF có thể tăng khi traffic, log retention, số lượng request hoặc dữ liệu lưu trữ tăng. Vì vậy, estimate này được dùng làm baseline cho target architecture, không phải cam kết chi phí cố định.

*Cost-Optimized Implementation Mode*

Trong giai đoạn triển khai FCAJ và demo, nhóm sẽ vận hành theo cost-optimized mode để kiểm soát credit. Một số thành phần target như NAT Gateway theo nhiều AZ, RDS Multi-AZ, Auto Scaling backend/worker và các resource chạy 24/7 sẽ chỉ được bật khi cần kiểm thử hoặc trình diễn.

Chiến lược tối ưu chi phí gồm:

- Chạy single RDS hoặc Single-AZ khi chưa cần kiểm thử failover.
- Giảm hoặc tạm thời tránh chạy NAT Gateway đa AZ 24/7 trong giai đoạn lab.
- Bật backend/worker theo nhu cầu kiểm thử hoặc demo thay vì chạy liên tục.
- Chỉ bật Auto Scaling, Multi-AZ hoặc high availability path khi cần validate.
- Dùng AWS Billing, Cost Explorer và AWS Budgets để theo dõi credit, cảnh báo chi phí và cập nhật estimate khi kiến trúc thay đổi.

*Free Tier/Credit Positioning*

Target architecture không được thiết kế để nằm hoàn toàn trong AWS Free Tier. Free Tier và AWS credits chỉ được dùng để giảm chi phí triển khai và trình diễn trong giai đoạn FCAJ, không phải baseline chính cho production-style architecture. AWS Pricing Calculator chỉ cung cấp ước tính; chi phí thực tế sẽ phụ thuộc vào usage, traffic, thời gian chạy resource, dữ liệu lưu trữ, cấu hình cuối cùng và thuế/phí phát sinh nếu có.

### 7. Đánh giá rủi ro
*Ma trận rủi ro*

- *Chi phí vượt dự kiến*: ảnh hưởng cao, xác suất trung bình do NAT Gateway, RDS, EC2, ALB và WAF có chi phí theo giờ hoặc usage.
- *Queue backlog*: ảnh hưởng cao, xác suất trung bình nếu attack campaign tạo nhiều event hơn tốc độ xử lý của worker.
- *Sai lệch schema hoặc feature*: ảnh hưởng cao, xác suất trung bình nếu collector, backend và AI models không thống nhất event contract.
- *False positive/false negative từ AI*: ảnh hưởng cao, xác suất trung bình do model có thể thiếu dữ liệu hoặc chưa bao phủ đầy đủ attack pattern.
- *Cấu hình WAF/ALB sai*: ảnh hưởng trung bình, xác suất trung bình vì có thể làm hỏng API/WebSocket routing.
- *RDS failover hoặc storage issue*: ảnh hưởng cao, xác suất thấp đến trung bình.
- *Lộ secret hoặc credential*: ảnh hưởng cao, xác suất thấp nếu secrets bị đưa vào tài liệu, screenshot hoặc source code.
- *Dashboard realtime mất kết nối*: ảnh hưởng trung bình, xác suất trung bình do WebSocket timeout, routing hoặc scaling.

*Chiến lược giảm thiểu*

- Tách backend tier và worker tier để scale theo đúng tín hiệu: request/CPU cho backend, queue backlog/message age cho worker.
- Dùng SQS và DLQ để hấp thụ burst traffic, retry message lỗi và hỗ trợ điều tra root cause.
- Dùng schema validation và event normalization tại backend ingestion API trước khi đưa message vào SQS.
- Dùng model output contract thống nhất để Fusion Layer xử lý completed, not_applicable, not_available hoặc failed một cách minh bạch.
- Dùng Secrets Manager, KMS và IAM least privilege để giảm rủi ro credential.
- Dùng AWS Budgets/Cost Explorer và cost-optimized mode trong giai đoạn demo.
- Dùng CloudWatch, CloudTrail và SNS để theo dõi vận hành, audit và cảnh báo.

*Kế hoạch dự phòng*

- Nếu chi phí lab tăng nhanh, chuyển sang single-instance/single-AZ mode, tắt NAT/worker/backend khi không demo và chỉ bật HA path trong thời gian kiểm thử.
- Nếu worker backlog tăng, tăng số worker tạm thời hoặc giảm tốc độ replay/collector.
- Nếu WebSocket gặp lỗi qua CloudFront/ALB, fallback sang REST polling cho dashboard trong demo.
- Nếu AI model chưa sẵn sàng, Fusion Layer vẫn có thể xử lý rule-based evidence hoặc model subset với trạng thái rõ ràng.

### 8. Kết quả kỳ vọng
*Cải tiến kỹ thuật*

Hệ thống kỳ vọng tạo được một SOC dashboard realtime có khả năng nhận event từ Local Security Lab, xử lý qua backend ingestion API, queue bất đồng bộ bằng SQS, chạy AI multi-model inference, tổng hợp bởi Fusion Layer và lưu final alert vào RDS/S3. Dashboard sẽ hiển thị KPI, alert feed, evidence, decision flow, MITRE mapping và trạng thái hệ thống.

*Giá trị dài hạn*

Dự án tạo nền tảng thực hành cho security monitoring, cloud architecture, AI security analytics và incident response. Kiến trúc tách backend/worker giúp mở rộng theo workload thực tế, RDS/S3 cung cấp dữ liệu bền vững cho phân tích, còn CloudWatch/CloudTrail/SNS hỗ trợ vận hành và audit. Cách trình bày target architecture cùng cost-optimized implementation mode cũng giúp nhóm chứng minh năng lực thiết kế kiến trúc có cân bằng giữa reliability, security, scalability và cost optimization.
