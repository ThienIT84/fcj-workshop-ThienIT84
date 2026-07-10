---
title: "Giới thiệu"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---



## Tuyên bố vấn đề

Trong các hệ thống giám sát bảo mật truyền thống, log thường được sinh ra từ nhiều nguồn khác nhau như firewall, IDS/IPS, network sensor hoặc server. Nếu không có một pipeline tập trung, việc thu thập và xử lý log sẽ gặp nhiều khó khăn:

- Log bị phân tán ở nhiều máy hoặc nhiều công cụ khác nhau.
- Việc gửi log từ Local Lab lên Cloud chưa được tự động hóa.
- Backend có thể bị quá tải nếu xử lý trực tiếp toàn bộ event.
- Raw evidence khó được lưu trữ đầy đủ để phục vụ audit hoặc phân tích lại.
- Dashboard không có dữ liệu tập trung để hiển thị cảnh báo.
- Hệ thống khó mở rộng nếu không tách riêng tầng nhận event, tầng xử lý và tầng lưu trữ.

Workshop này giải quyết vấn đề bằng cách xây dựng một kiến trúc có khả năng:

**Thu thập log/evidence:** Nhận dữ liệu từ Local Lab thông qua Zeek Collector hoặc Tailer.

**Tiếp nhận event:** EC2 Backend nhận request từ collector hoặc dashboard.

**Xử lý bất đồng bộ:** Amazon SQS tách luồng ingest và luồng xử lý để backend không bị quá tải.

**Lưu trữ raw evidence:** Amazon S3 Data Bucket lưu dữ liệu gốc phục vụ audit và xử lý lại khi cần.

**Lưu trữ kết quả xử lý:** Amazon RDS PostgreSQL lưu alert, event metadata và kết quả phân tích.

**Hiển thị cảnh báo:** React Dashboard hiển thị alert cho admin hoặc SOC analyst thông qua API/WebSocket.

## Kiến trúc giải pháp

Hệ thống được thiết kế theo kiến trúc hybrid giữa Local Security Lab và AWS Cloud. Local Lab tạo log và evidence, sau đó collector gửi dữ liệu lên backend chạy trên EC2. Backend tiếp nhận event, ghi raw evidence vào S3 và gửi message vào SQS để worker xử lý bất đồng bộ.

Sau khi xử lý, dữ liệu được lưu vào RDS PostgreSQL và hiển thị lên dashboard thông qua API hoặc WebSocket.

Flow tổng quát của hệ thống:

```text
Local Lab
-> Zeek Collector / Tailer
-> EC2 Backend
-> SQS + S3 Data Bucket
-> Worker / Fusion
-> RDS PostgreSQL
-> API / WebSocket
-> React Dashboard
```

### Local Lab

Local Lab là nơi sinh dữ liệu đầu vào cho hệ thống SOC. Trong mô hình này, Local Lab gồm attacker, firewall/router, victim machine, Zeek sensor và collector/tailer.

Các thành phần chính:

- Kali Linux mô phỏng attacker hoặc tạo traffic kiểm thử.
- pfSense đóng vai trò firewall/router.
- Victim machine tạo traffic để sensor quan sát.
- Zeek sensor phân tích network traffic và sinh log.
- Zeek Collector/Tailer đọc log và gửi evidence lên cloud backend.

### Frontend Interface

Người dùng truy cập hệ thống thông qua dashboard trên trình duyệt web.

Luồng truy cập dashboard:

```text
Admin / Analyst
-> HTTPS Dashboard
-> CloudFront
-> S3 Static Frontend
```

Frontend được triển khai dưới dạng static web application. Amazon S3 lưu static assets, CloudFront phân phối nội dung qua CDN, còn WAF có thể được dùng để lọc request độc hại ở tầng edge nếu đã cấu hình.

Dashboard giúp admin hoặc analyst theo dõi:

- Danh sách alert.
- Mức độ nghiêm trọng.
- Nguồn log.
- Thời gian phát hiện.
- Chi tiết event.
- Trạng thái xử lý.

### Request Handling

Request từ dashboard hoặc collector sẽ được gửi về backend API.

Trong kiến trúc mục tiêu, request có thể đi qua Application Load Balancer và Target Group trước khi đến backend EC2. Backend chạy FastAPI để xử lý REST API, WebSocket hoặc các endpoint ingest event.

Các thành phần chính:

- Application Load Balancer định tuyến traffic đến backend.
- Target Group quản lý các EC2 backend instance.
- EC2 Backend chạy FastAPI API service.
- Auto Scaling Group có thể mở rộng số lượng backend instance khi cần.

Ở giai đoạn hiện tại, nếu ALB hoặc Auto Scaling Group chưa được cấu hình hoàn chỉnh, backend có thể được kiểm tra trực tiếp qua EC2 để hoàn thành core flow trước.

### Backend Processing

Backend Processing Layer chịu trách nhiệm xử lý event sau khi backend nhận dữ liệu từ Local Lab.

Luồng xử lý chính:

```text
EC2 Backend
-> S3 Data Bucket
-> SQS Queue
-> Worker / Fusion
-> RDS PostgreSQL
```

EC2 Backend sẽ nhận event, validate payload, sinh `event_id`, ghi raw evidence vào S3 Data Bucket và gửi message vào SQS. Worker hoặc Fusion component đọc message từ SQS, xử lý event và lưu kết quả vào RDS PostgreSQL.

Cách thiết kế này giúp hệ thống:

- Giảm tải cho backend API.
- Tách riêng quá trình nhận event và xử lý event.
- Hạn chế mất dữ liệu khi worker lỗi.
- Có thể mở rộng worker độc lập.
- Lưu lại raw evidence để phục vụ audit.

### Data Layer

Data Layer gồm các service lưu trữ dữ liệu gốc và dữ liệu đã xử lý.

Amazon S3 được dùng để lưu raw evidence từ Local Lab. Ví dụ một object key có thể có dạng:

```text
raw/zeek/http/2026/07/06/evt-xxxx.json
```

Amazon RDS PostgreSQL được dùng để lưu:

- Event metadata.
- Alert.
- Detection result.
- Severity.
- Timestamp.
- Source log type.
- Processing status.

Trong giai đoạn demo, RDS PostgreSQL có thể được triển khai ở chế độ Single-AZ để tối ưu chi phí. Multi-AZ là phần mở rộng production-style và không được claim là đã triển khai nếu chưa bật thật.

### Network and Security

Hệ thống được triển khai trong một AWS Region và một VPC riêng. Bên trong VPC, các subnet được chia theo vai trò để tách biệt public traffic, backend runtime và database.

Các thành phần network chính:

- VPC làm mạng riêng cho toàn bộ hệ thống.
- Public subnet chứa NAT Gateway hoặc ALB nếu có.
- Private subnet chứa backend EC2 hoặc worker.
- Private DB subnet chứa RDS PostgreSQL.
- Internet Gateway cho public subnet kết nối Internet.
- NAT Gateway cho private subnet đi outbound Internet nếu có.
- Security Group kiểm soát inbound và outbound traffic.
- VPC Endpoint cho phép truy cập S3 mà không cần đi qua Internet.

Security baseline của hệ thống gồm:

- IAM Role cho EC2 Backend.
- Secrets Manager để lưu thông tin nhạy cảm.
- KMS để mã hóa secret hoặc dữ liệu nếu được cấu hình.
- Security Group giới hạn SSH, backend port và database port.
- Không hard-code AWS access key trong source code hoặc `.env`.

### Monitoring and Notification

Monitoring giúp nhóm kiểm tra trạng thái backend, pipeline và các lỗi runtime.

Các thành phần chính:

- CloudWatch dùng để theo dõi log và metric.
- CloudTrail ghi lại API activity trong AWS account.
- SNS có thể dùng để gửi cảnh báo qua Email/SMS.

Nếu backend lỗi, nhóm có thể kiểm tra log trực tiếp trên EC2 bằng:

```bash
sudo journalctl -u socai-backend -n 80 --no-pager
```

Nếu đã cấu hình CloudWatch Logs, log backend có thể được đẩy lên CloudWatch để theo dõi tập trung.

## Sơ đồ kiến trúc

![Hybrid SOC Architecture](/images/5-Workshop/5.1-Overview-and-Architecture/w-trang-01-architecture-overview.jpg)

## Công nghệ chính

Trong workshop này, bạn sẽ làm việc với các dịch vụ AWS và công nghệ chính sau:

**Amazon EC2:** Chạy FastAPI Backend, API service, WebSocket hoặc worker logic.

**Amazon VPC:** Cung cấp network riêng cho hệ thống, bao gồm subnet, route table, Internet Gateway, NAT Gateway và Security Group.

**Amazon S3:** Lưu static frontend và raw evidence từ Local Lab.

**Amazon SQS:** Message queue dùng để xử lý event bất đồng bộ giữa backend và worker.

**Amazon RDS PostgreSQL:** Lưu event metadata, alert và kết quả xử lý.

**AWS IAM:** Quản lý quyền truy cập cho EC2 Backend thông qua IAM Role.

**AWS Secrets Manager:** Lưu thông tin nhạy cảm như database credential.

**AWS KMS:** Hỗ trợ mã hóa secret hoặc dữ liệu nếu hệ thống yêu cầu.

**Amazon CloudFront:** Phân phối frontend dashboard qua CDN.

**AWS WAF:** Bảo vệ frontend hoặc API khỏi các request độc hại nếu được cấu hình.

**Application Load Balancer:** Định tuyến traffic đến backend EC2 thông qua Target Group.

**Auto Scaling Group:** Mở rộng số lượng EC2 Backend trong kiến trúc production-style.

**Amazon CloudWatch:** Theo dõi log, metric và trạng thái hoạt động của backend.

**AWS CloudTrail:** Ghi nhận các hoạt động API trong AWS account.

**Amazon SNS:** Gửi cảnh báo qua Email/SMS nếu hệ thống bật notification.

## Trạng thái triển khai hiện tại

Một số thành phần trong sơ đồ là kiến trúc mục tiêu hoặc phần mở rộng production-style. Trong giai đoạn hiện tại, workshop ưu tiên hoàn thành core flow trước:

```text
Local Lab
-> EC2 Backend
-> SQS / S3
-> Worker / RDS
-> Dashboard
```

Các thành phần như ALB, Target Group, Auto Scaling Group, CloudTrail và SNS chỉ nên được ghi là phần mở rộng nếu chưa có evidence cấu hình thật trên AWS Console.

Backend hiện đang được cập nhật source để bổ sung boto3 integration cho S3 và SQS. Vì vậy, section sau sẽ tập trung kiểm tra EC2 runtime, systemd service, endpoint `/health` và đặt  cho endpoint `/api/events/http/async` đến khi source mới sẵn sàng.
