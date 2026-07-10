---
title: "NAT Gateway"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 5.3.5. </b> "
---

## Giới thiệu

NAT Gateway cho phép các instance trong **private subnet** đi outbound Internet, ví dụ update package, pull source code hoặc gọi external endpoint.

Trong hệ thống **Hybrid SOC trên AWS**, NAT Gateway được dùng để hỗ trợ Backend EC2 hoặc Worker trong private subnet có thể truy cập Internet khi cần, nhưng Internet không thể chủ động truy cập ngược vào private instance.

## Bước 1: Vào trang NAT Gateways

Trên AWS Console, mở dịch vụ **VPC**, sau đó chọn **NAT gateways** ở menu bên trái.

Trong project hiện tại, NAT Gateway đã được tạo với tên:

```text
socai-dev-nat-a
```

Hình dưới đây cho thấy NAT Gateway đang ở trạng thái `Available`.

![NAT Gateway List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-05-01-nat-gateway-list.jpg)

## Bước 2: Kiểm tra thông tin NAT Gateway

Chọn NAT Gateway `socai-dev-nat-a` để kiểm tra thông tin chi tiết.

Thông tin NAT Gateway hiện tại:

```text
NAT Gateway name: socai-dev-nat-a
NAT Gateway ID: nat-0a7e6ff8f16b60be7
Connectivity type: Public
State: Available
VPC: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
Subnet: social-dev-public-subnet-a
Subnet ID: subnet-09ac666babe09a780
Primary Private IPv4: 10.20.2.27
Primary Public IPv4: 47.131.186.17
```

NAT Gateway đang nằm trong public subnet `social-dev-public-subnet-a`, đây là cấu hình đúng vì NAT Gateway cần có đường ra Internet thông qua Internet Gateway.

![NAT Gateway Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-05-02-nat-gateway-detail.jpg)

## Bước 3: Xác định private subnet sử dụng NAT Gateway

Các private app subnet của project gồm:

```text
social-dev-private-subnet-a
social-dev-private-subnet-b
```

Hai subnet này được dùng cho Backend EC2 hoặc Worker trong private layer.

Nếu các instance trong private subnet cần truy cập Internet, route table của private subnet cần có route đi qua NAT Gateway.

Route cần kiểm tra ở private route table:

```text
Destination: 0.0.0.0/0
Target: nat-0a7e6ff8f16b60be7
```

Hình dưới đây cho thấy các private app subnet thuộc VPC `socai-dev-vpc`.

![Private App Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-05-03-private-app-subnets.jpg)

## Vai trò NAT Gateway trong workshop

| Thành phần | Vai trò |
|---|---|
| NAT Gateway | Cho private subnet đi outbound Internet |
| Public subnet | Nơi đặt NAT Gateway |
| Private app subnet | Nơi đặt Backend EC2 hoặc Worker |
| Internet Gateway | Cho NAT Gateway đi ra Internet |
| Route table | Điều hướng traffic từ private subnet đến NAT Gateway |

NAT Gateway giúp backend hoặc worker trong private subnet có thể truy cập Internet mà không cần public IP trực tiếp.

## Kết luận

Sau bước này, người thực hiện cần xác nhận:

- NAT Gateway `socai-dev-nat-a` đã được tạo.
- NAT Gateway đang ở trạng thái `Available`.
- NAT Gateway thuộc VPC `socai-dev-vpc`.
- NAT Gateway được đặt trong public subnet `social-dev-public-subnet-a`.
- Private app subnet có thể dùng NAT Gateway để outbound Internet nếu route table đã trỏ đúng.
- Route table private cần được kiểm tra để đảm bảo có route `0.0.0.0/0` đến NAT Gateway.

Phần tiếp theo sẽ kiểm tra **Security Groups**.
