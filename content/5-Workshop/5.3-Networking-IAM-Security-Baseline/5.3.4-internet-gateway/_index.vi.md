---
title: "Internet Gateway"
date: 2026-07-07
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---


## Giới thiệu

Internet Gateway cho phép các tài nguyên trong **public subnet** kết nối ra Internet. Trong hệ thống **Hybrid SOC trên AWS**, Internet Gateway thường được dùng cho public subnet chứa ALB, NAT Gateway hoặc EC2 public trong giai đoạn demo.

Sau khi đã tạo VPC, subnet và route table, cần kiểm tra Internet Gateway đã được gắn đúng với VPC `socai-dev-vpc` hay chưa.

## Bước 1: Vào trang Internet Gateways

Trên AWS Console, mở dịch vụ **VPC**, sau đó chọn **Internet gateways** ở menu bên trái.

Nếu chưa có Internet Gateway cho project, chọn **Create internet gateway**.

![Create Internet Gateway](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-04-01-create-internet-gateway.jpg)

## Bước 2: Tạo Internet Gateway

Ở màn hình tạo Internet Gateway, nhập tên:
![Internet Gateway Settings](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-04-02-internet-gateway-settings.jpg)

Sau đó chọn **Create internet gateway**.



## Bước 3: Attach Internet Gateway vào VPC

Sau khi tạo xong, chọn Internet Gateway vừa tạo và attach vào VPC của project:

```text
VPC name: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
```

Internet Gateway chỉ hoạt động khi đã được attach vào VPC.


## Bước 4: Kiểm tra Internet Gateway sau khi attach

Sau khi attach xong, kiểm tra Internet Gateway đang ở trạng thái attached với VPC `socai-dev-vpc`.

Thông tin cần kiểm tra:

```text
Internet Gateway name: socai-dev-igw
State: Attached
VPC: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
```

![Internet Gateway Attached](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-04-03-internet-gateway-attached.jpg)


## Vai trò trong workshop

Internet Gateway được dùng cho public subnet. Public route table sẽ trỏ route Internet về Internet Gateway.

Route cần có trong public route table:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Public subnet có thể dùng cho:

- ALB nếu cấu hình ở phase sau.
- NAT Gateway nếu private subnet cần outbound Internet.
- EC2 public nếu cần demo nhanh.

## Kết luận

Sau bước này, người thực hiện cần đảm bảo:

- Internet Gateway đã được tạo.
- Internet Gateway đã attach vào VPC `socai-dev-vpc`.
- Public route table có thể trỏ route `0.0.0.0/0` đến Internet Gateway.
- Internet Gateway chỉ phục vụ public subnet.
- Backend public nếu có chỉ là demo mode, không phải cấu hình production-style.

Phần tiếp theo sẽ kiểm tra **NAT Gateway**.
