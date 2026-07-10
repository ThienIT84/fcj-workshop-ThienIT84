---
title: "Tổng quan VPC"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---


## Giới thiệu

VPC là mạng riêng của hệ thống **Hybrid SOC trên AWS**. Đây là nền tảng network để triển khai Backend EC2, Worker, RDS PostgreSQL, Security Group và các thành phần network liên quan.

Trong workshop này, chúng ta tạo một VPC riêng cho project với tên:

```text
socai-dev-vpc
```

VPC này sẽ được dùng cho các bước tiếp theo như tạo subnet, route table, Internet Gateway, NAT Gateway và Backend EC2.


## Bước 1: Tạo VPC

Trên AWS Console, mở dịch vụ **VPC**, vào trang **Your VPCs** và chọn **Create VPC** như hình bên dưới.

![Create VPC Button](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-01-create-vpc-button.jpg)


## Bước 2: Cấu hình VPC

Ở màn hình **Create VPC**, cấu hình như sau:

Trong workshop này, chọn **VPC only** để tự cấu hình riêng các thành phần phía sau như subnet, route table, Internet Gateway và NAT Gateway.

Dải CIDR `10.20.0.0/16` đủ để chia thành nhiều subnet cho public layer, app layer và database layer.

![Create VPC Settings](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-02-create-vpc-settings.jpg)

Sau khi kiểm tra đúng thông tin, chọn **Create VPC**.


## Bước 3: Kiểm tra VPC sau khi tạo

Sau khi tạo xong, quay lại trang **Your VPCs** và kiểm tra VPC vừa tạo.

Thông tin cần thấy:

![VPC List Created](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-03-vpc-list-created.jpg)

## Bước 4: Kiểm tra chi tiết VPC

Chọn VPC `socai-dev-vpc` để kiểm tra thông tin chi tiết.

Cần xác nhận các thông tin chính:

![VPC Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-04-vpc-detail.jpg)


## Vai trò của VPC trong workshop

Trong kiến trúc Hybrid SOC, VPC được dùng để tách các lớp network chính:

| Lớp network | Vai trò |
|---|---|
| Public subnet | Dùng cho ALB hoặc NAT Gateway nếu có cấu hình |
| Private app subnet | Dùng cho Backend EC2 hoặc Worker |
| Private DB subnet | Dùng cho RDS PostgreSQL |

Việc tạo VPC riêng giúp hệ thống dễ quản lý hơn, tránh dùng chung network với project khác và thuận tiện cho việc kiểm soát route table, security group và database access.

## Lưu ý

Ở bước này, chúng ta chỉ tạo **VPC only**, nên các thành phần sau chưa được tạo tự động:

```text
Subnets
Internet Gateway
NAT Gateway
Custom Route Tables
Security Groups riêng cho backend
```

Các thành phần này sẽ được cấu hình ở những bước tiếp theo.

Trước khi đưa screenshot vào workshop, cần che hoặc crop các thông tin nhạy cảm như Owner ID hoặc account ID nếu xuất hiện trong ảnh.


## Kết luận

Sau bước này, người thực hiện cần đảm bảo:

- VPC `socai-dev-vpc` đã được tạo.
- IPv4 CIDR là `10.20.0.0/16`.
- VPC đang ở trạng thái `Available`.
- Tenancy là `default`.
- DNS resolution và DNS hostnames đã được bật.
- VPC sẵn sàng để tiếp tục tạo subnet, route table và security baseline.

Phần tiếp theo sẽ kiểm tra **Subnets**.
