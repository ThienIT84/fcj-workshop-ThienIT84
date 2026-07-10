---
title: "Security Groups"
date: 2026-07-08
weight: 6
chapter: false
pre: " <b> 5.3.6. </b> "
---

## Giới thiệu

Security Group hoạt động như firewall cho các tài nguyên trong AWS. Trong hệ thống **Hybrid SOC trên AWS**, Security Group được dùng để kiểm soát traffic đi vào và đi ra khỏi các thành phần như ALB, Backend EC2, AI Worker và RDS PostgreSQL.

Sau khi đã kiểm tra VPC, subnet, route table, Internet Gateway và NAT Gateway, bước tiếp theo là kiểm tra các Security Group thuộc VPC `socai-dev-vpc`.

## Bước 1: Kiểm tra danh sách Security Groups

Trên AWS Console, mở dịch vụ **EC2**, sau đó chọn **Security Groups** trong mục **Network & Security**.

Trong workshop này, chỉ kiểm tra các Security Group thuộc VPC của project:

```text
VPC ID: vpc-04c78e55ae4ceec8a
VPC Name: socai-dev-vpc
```

Danh sách Security Group hiện tại trong project gồm:

| Security Group name | Security Group ID | Vai trò |
|---|---|---|
| `socai-dev-alb-sg` | `sg-0e6d896dc7e2f9d56` | Security Group cho ALB |
| `socai-dev-ai-worker-sg` | `sg-0c769c2c9935d039f` | Security Group cho AI Worker |
| `socai-dev-rds-sg` | `sg-0f8e019c2101c8f08` | Security Group cho RDS PostgreSQL |
| `socai-dev-backend-sg` | `sg-0de2d07c6a356954a` | Security Group cho Backend EC2 |
| `default` | `sg-045764b4d04e69246` | Default Security Group trong VPC project |

Không sử dụng Security Group thuộc default VPC:

```text
VPC ID: vpc-0bc9182791a842901
```

Hình dưới đây cho thấy các Security Group thuộc VPC `socai-dev-vpc`.

![Security Groups List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-06-01-security-groups-list.jpg)

## Bước 2: Kiểm tra Backend Security Group

Backend EC2 sử dụng Security Group:

```text
Security Group name: socai-dev-backend-sg
Security Group ID: sg-0de2d07c6a356954a
Description: FastAPI backend and WebSocket instances
VPC ID: vpc-04c78e55ae4ceec8a
```

Security Group này dùng để kiểm soát traffic vào Backend EC2.

Backend Security Group hiện tại có:

```text
Inbound rules count: 1
Outbound rules count: 1
```

## Bước 3: Kiểm tra Inbound Rules

Inbound rules quyết định traffic nào được phép đi vào Backend EC2.

Trong project hiện tại, Backend Security Group có 1 inbound rule:

```text
Type: Custom TCP
Protocol: TCP
Port range: 8000
Source: Security Group của ALB
```

Port `8000` là port backend FastAPI đang sử dụng. Source của rule là một Security Group khác, không phải `0.0.0.0/0`, nên backend không mở trực tiếp port `8000` cho toàn bộ Internet.

Điều này phù hợp với mô hình:

```text
Client
-> ALB
-> Backend EC2 port 8000
```

Hình dưới đây cho thấy inbound rule của `socai-dev-backend-sg`.

![Backend Security Group Inbound](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-06-02-backend-sg-inbound.jpg)

## Bước 4: Kiểm tra Outbound Rules

Outbound rules quyết định Backend EC2 được phép gọi ra ngoài hoặc gọi đến service khác.

Trong project hiện tại, Backend Security Group có 1 outbound rule:

```text
Type: All traffic
Protocol: All
Port range: All
Destination: 0.0.0.0/0
```

Rule này cho phép backend gửi traffic outbound ra ngoài. Backend có thể dùng outbound traffic để gọi các service hoặc endpoint cần thiết như SQS, S3, Secrets Manager, CloudWatch hoặc RDS nếu route table, IAM Role và Security Group phía đích cho phép.

Trong môi trường demo, outbound mở rộng như trên giúp dễ kiểm thử. Ở môi trường production, có thể siết chặt outbound theo đúng destination và port cần thiết.

Hình dưới đây cho thấy outbound rule của `socai-dev-backend-sg`.

![Backend Security Group Outbound](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-06-03-backend-sg-outbound.jpg)

## Vai trò các Security Groups trong project

| Security Group | Vai trò |
|---|---|
| `socai-dev-alb-sg` | Nhận traffic từ client và chuyển tiếp về backend |
| `socai-dev-backend-sg` | Cho phép backend nhận traffic từ ALB qua port `8000` |
| `socai-dev-ai-worker-sg` | Dùng cho worker xử lý event |
| `socai-dev-rds-sg` | Bảo vệ RDS PostgreSQL |

Backend Security Group hiện tại không thể hiện rule SSH port `22`. Vì vậy, trong workshop này không claim backend đang mở SSH nếu chưa có evidence thật.

## Demo mode và ALB mode

Trong kiến trúc hiện tại, inbound của backend đang đi theo hướng ALB mode:

```text
Client
-> ALB
-> Backend EC2 port 8000
```

Với cấu hình này, backend port `8000` chỉ nên nhận traffic từ Security Group của ALB.

Không nên mở backend port `8000` trực tiếp cho:

```text
0.0.0.0/0
```

nếu backend đã nằm sau ALB.

## Kết luận

Sau bước này, người thực hiện cần đảm bảo:

- Backend EC2 dùng Security Group `socai-dev-backend-sg`.
- Backend nhận traffic qua port `8000`.
- Source inbound là Security Group khác, phù hợp với mô hình đi qua ALB.
- Outbound hiện cho phép `All traffic` đến `0.0.0.0/0`.
- Không sử dụng Security Group thuộc default VPC.
- Không claim SSH port `22` nếu chưa có rule và screenshot thật.
- RDS port `5432` không được public ra Internet.

Phần tiếp theo sẽ kiểm tra **IAM Role for EC2 Backend**.
