---
title: "Các điều kiện tiên quyết và Quy tắc đặt tên"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---


## Giới thiệu

Trước khi triển khai hệ thống **Hybrid SOC trên AWS**, người thực hiện cần chuẩn bị đúng môi trường AWS, công cụ local và quy ước đặt tên resource.

Phần này giúp workshop được triển khai thống nhất, tránh các lỗi thường gặp như tạo resource sai region, thiếu quyền IAM, thiếu công cụ local hoặc để lộ secret trong tài liệu.


## AWS Account và Region

Người thực hiện cần có một **AWS account** dùng cho workshop và có quyền truy cập AWS Console.

Các service cần có quyền tạo hoặc kiểm tra gồm:

- VPC
- EC2
- IAM Role
- S3
- SQS
- RDS
- Secrets Manager
- CloudWatch

Workshop sử dụng cùng một region cho toàn bộ resource:

Trong workshop này, region được sử dụng là:

```text
ap-southeast-1
```

Tất cả resource nên được tạo cùng region để tránh lỗi endpoint, lỗi kết nối và khó quản lý chi phí.

Cách kiểm tra region:

```text
AWS Console
-> Thanh top-right
-> Region selector
-> Chọn Asia Pacific (Singapore) - ap-southeast-1
```

Hình dưới đây xác nhận AWS Console đang được đặt đúng region **Asia Pacific (Singapore) - ap-southeast-1**.

<img src="/images/5-Workshop/5.2-Prerequisites-and-Naming/w-trang-01-aws-region.jpg" alt="AWS Region" style="max-width: 500px !important; width: auto !important; height: auto !important; display: block !important; margin: 1.5rem auto !important;">


## IAM Permission Check

Trước khi cấu hình backend, cần kiểm tra quyền IAM tổng quan.

Cách vào IAM Dashboard:

```text
AWS Console
-> Search "IAM"
-> IAM Dashboard
-> Users / Roles / Policies
```

Người thực hiện cần có quyền tạo hoặc đọc các service sau:

```text
VPC, EC2, IAM Role, S3, SQS, RDS, Secrets Manager, CloudWatch
```

Nếu không có quyền tạo IAM Role, cần nhờ owner account hoặc admin tạo role trước.

IAM Role đề xuất cho backend:

```text
socai-backend-role
```

Role này sẽ được gắn vào EC2 Backend để backend có thể gọi S3, SQS và Secrets Manager mà không cần hard-code AWS access key trong source code hoặc file `.env`.

Hình dưới đây cho thấy IAM Dashboard.

![IAM Dashboard](/images/5-Workshop/5.2-Prerequisites-and-Naming/w-trang-02-iam-dashboard.jpg)


## Local Tools

Máy local cần có các công cụ cơ bản để thao tác source code, kiểm tra AWS CLI và kết nối EC2.

Các công cụ cần chuẩn bị:

- Git
- Python
- AWS CLI
- SSH client
- Node.js / pnpm nếu build frontend
- Local Lab nếu chạy end-to-end

Kiểm tra bằng command:

```bash
aws --version
python --version
git --version
```

Kết quả kiểm tra trên máy local:

```text
aws-cli/2.35.15
Python 3.12.10
git version 2.54.0.windows.1
```

Hình dưới đây chứng minh máy local đã có đủ các công cụ cơ bản để tiếp tục workshop.

![Local Tools Version](/images/5-Workshop/5.2-Prerequisites-and-Naming/w-trang-03-local-tools-version.jpg)


## Naming Convention

Workshop sử dụng quy ước đặt tên thống nhất để dễ quản lý resource trong AWS Console.

| Resource | Tên đề xuất |
|---|---|
| VPC | `socai-vpc` |
| EC2 Backend | `socai-backend-ec2` |
| IAM Role Backend | `socai-backend-role` |
| Security Group Backend | `socai-backend-sg` |
| S3 Static Bucket | `socai-static-fe-<account>-<region>` |
| S3 Data Bucket | `socai-data-bucket-<account>-<region>` |
| SQS Main Queue | `socai-events-queue` |
| SQS DLQ | `socai-events-dlq` |
| RDS | `socai-postgres` |

Quy ước này giúp nhóm dễ tìm resource, dễ bàn giao thông tin và tránh nhầm lẫn giữa các thành phần như EC2 Backend, S3 bucket, SQS queue và RDS database.


##  Convention

Không đưa giá trị thật hoặc secret thật vào workshop. Các giá trị thay đổi theo môi trường cần được viết bằng .

Các  chính:

```text
<AWS_REGION>
<EC2_PUBLIC_DNS>
<SQS_QUEUE_URL>
<S3_DATA_BUCKET>
<RDS_SECRET_ID>
<RDS_ENDPOINT>
```

Ví dụ test backend health endpoint:

```bash
curl -i http://<EC2_PUBLIC_DNS>:8000/health
```

Ví dụ biến môi trường cho backend:

```text
AWS_REGION=<AWS_REGION>
SQS_QUEUE_URL=<SQS_QUEUE_URL>
S3_DATA_BUCKET=<S3_DATA_BUCKET>
RDS_SECRET_ID=<RDS_SECRET_ID>
```


## Secret Safety

Không đưa các thông tin nhạy cảm vào markdown, command hoặc screenshot.

Không đưa các thông tin sau vào workshop:

- AWS access key
- AWS secret key
- DB password
- Private key `.pem`
- Token hoặc API key
- Secret value
- Nội dung `.env` chưa che
- Full account ID nếu không cần

Nếu cần minh họa cấu hình, chỉ dùng dạng đã che:

```text
DATABASE_URL=<>
SQS_QUEUE_URL=<>
S3_DATA_BUCKET=socai-data-bucket-...
RDS_SECRET_ID=<>
```


## Kết luận

Sau phần này, người thực hiện cần đảm bảo:

- Đã có AWS account dùng cho workshop.
- Đã chọn đúng region `ap-southeast-1`.
- Có quyền kiểm tra hoặc tạo các service cần thiết.
- Máy local có Git, Python, AWS CLI và SSH client.
- Resource được đặt tên theo cùng convention.
- Không đưa secret thật vào workshop.

Phần tiếp theo sẽ cấu hình **Networking, IAM và Security Baseline** cho Backend EC2.
