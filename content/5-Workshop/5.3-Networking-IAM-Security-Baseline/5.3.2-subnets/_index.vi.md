---
title: "Subnets"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---


## Giới thiệu

Subnet dùng để chia VPC thành nhiều vùng mạng nhỏ theo từng vai trò. Trong hệ thống **Hybrid SOC trên AWS**, subnet giúp tách riêng public layer, backend app layer và database layer.

Sau khi đã tạo VPC `socai-dev-vpc`, chúng ta sẽ tạo và kiểm tra các subnet thuộc VPC này để chuẩn bị cho Backend EC2, NAT Gateway hoặc ALB, và RDS PostgreSQL.


## Bước 1: Vào trang Subnets

Trên AWS Console, mở dịch vụ **VPC**, chọn **Subnets** ở menu bên trái.

Sau đó chọn **Create subnet** để bắt đầu tạo subnet mới.

![Create Subnet Button](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-01-create-subnet-button.jpg)


## Bước 2: Chọn VPC cho subnet

Ở phần **VPC ID**, chọn VPC của project:

```text
socai-dev-vpc
```

Thông tin VPC:

```text
VPC ID: vpc-04c78e55ae4ceec8a
IPv4 CIDR: 10.20.0.0/16
```

Tất cả subnet trong phần này phải được tạo trong cùng VPC `socai-dev-vpc`.

Không sử dụng các subnet có CIDR `172.31.x.x` vì đó là subnet thuộc default VPC, không phải VPC của project.

![Select VPC for Subnet](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-02-setting-subnet.jpg)


## Bước 3: Kiểm tra Public Subnets

Public subnet dùng cho các thành phần cần public entrypoint như ALB hoặc NAT Gateway.

Trong project hiện tại, public subnet gồm:

```text
Subnet name: social-dev-public-subnet-a
Subnet ID: subnet-09ac666babe09a780
IPv4 CIDR: 10.20.0.0/20
```

```text
Subnet name: social-dev-public-subnet-b
Subnet ID: subnet-0f1b248ff01c3b297
IPv4 CIDR: 10.20.16.0/20
```

Public subnet thường sẽ được gắn với route table có route ra Internet Gateway:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

![Public Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-03-public-subnet-settings.jpg)


## Bước 4: Kiểm tra Private App Subnets

Private app subnet dùng cho Backend EC2 hoặc Worker trong kiến trúc production-style.

Trong project hiện tại, private app subnet gồm:

```text
Subnet name: social-dev-private-subnet-a
Subnet ID: subnet-0c510623f443f178e
IPv4 CIDR: 10.20.128.0/20
```

```text
Subnet name: social-dev-private-subnet-b
Subnet ID: subnet-071ded17acfbe50a0
IPv4 CIDR: 10.20.144.0/20
```

Backend EC2 trong private subnet không nên nhận traffic trực tiếp từ Internet. Nếu cần public access, request nên đi qua ALB hoặc CloudFront tùy kiến trúc cuối.

![Private App Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-04-private-app-subnet-settings.jpg)


## Bước 5: Kiểm tra Private DB Subnets

Private DB subnet dùng cho RDS PostgreSQL. Database không nên đặt trong public subnet hoặc route trực tiếp ra Internet Gateway.

Trong project hiện tại, private DB subnet gồm:

```text
Subnet name: socai-dev-private-db-subnet-a
Subnet ID: subnet-08fddc2ead9510633
IPv4 CIDR: 10.20.32.0/24
```

```text
Subnet name: socai-dev-private-db-subnet-b
Subnet ID: subnet-06a93d15ec51c7943
IPv4 CIDR: 10.20.33.0/24
```

RDS thường cần DB subnet group gồm ít nhất hai subnet ở hai Availability Zone khác nhau. Tuy nhiên, nếu project đang demo Single-AZ để giảm chi phí, cần ghi đúng trạng thái thật và không claim Multi-AZ nếu chưa bật.

![Private DB Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-05-private-db-subnet-settings.jpg)


## Bước 6: Kiểm tra danh sách Subnets

Sau khi tạo subnet, quay lại trang **Subnets** và kiểm tra danh sách subnet thuộc VPC của project.

Trong workshop này, chỉ sử dụng các subnet thuộc VPC:

```text
VPC ID: vpc-04c78e55ae4ceec8a
VPC CIDR: 10.20.0.0/16
```

Danh sách subnet hiện tại của project:

| Subnet name | Vai trò | Subnet ID | IPv4 CIDR |
|---|---|---|---|
| `social-dev-public-subnet-a` | Public subnet | `subnet-09ac666babe09a780` | `10.20.0.0/20` |
| `social-dev-public-subnet-b` | Public subnet | `subnet-0f1b248ff01c3b297` | `10.20.16.0/20` |
| `socai-dev-private-db-subnet-a` | Private DB subnet | `subnet-08fddc2ead9510633` | `10.20.32.0/24` |
| `socai-dev-private-db-subnet-b` | Private DB subnet | `subnet-06a93d15ec51c7943` | `10.20.33.0/24` |
| `social-dev-private-subnet-a` | Private app subnet | `subnet-0c510623f443f178e` | `10.20.128.0/20` |
| `social-dev-private-subnet-b` | Private app subnet | `subnet-071ded17acfbe50a0` | `10.20.144.0/20` |

Các subnet trên đều đang ở trạng thái:

```text
Available
```

Các subnet có CIDR `172.31.x.x` thuộc default VPC nên không được dùng trong project này.

![Subnet List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-06-subnet-list.jpg)


## Bước 7: Kiểm tra chi tiết Private App Subnet

Chọn private app subnet để kiểm tra thông tin chi tiết.

Ví dụ private app subnet đang kiểm tra:

```text
Subnet name: social-dev-private-subnet-a
Subnet ID: subnet-0c510623f443f178e
IPv4 CIDR: 10.20.128.0/20
Availability Zone: apse1-az2 (ap-southeast-1a)
State: Available
VPC: socai-dev-vpc
Auto-assign public IPv4 address: No
Route table: socai-dev-rt-private-a
```

Private app subnet dùng cho Backend EC2 hoặc Worker. Ở kiến trúc production-style, subnet này không nên public trực tiếp ra Internet.

![Private App Subnet Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-07-private-app-subnet-detail.jpg)


## Bước 8: Kiểm tra chi tiết Private DB Subnet

Chọn private DB subnet để kiểm tra thông tin chi tiết.

Ví dụ private DB subnet đang kiểm tra:

```text
Subnet name: socai-dev-private-db-subnet-a
Subnet ID: subnet-08fddc2ead9510633
IPv4 CIDR: 10.20.32.0/24
Availability Zone: apse1-az2 (ap-southeast-1a)
State: Available
VPC: socai-dev-vpc
Auto-assign public IPv4 address: No
Route table: socai-dev-rt-db
```

Private DB subnet dùng cho RDS PostgreSQL. Database subnet cần được giữ private để tránh public exposure.

![DB Subnet Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-08-db-subnet-detail.jpg)


## Vai trò subnet trong workshop

| Loại subnet | Vai trò |
|---|---|
| Public subnet | Dùng cho ALB hoặc NAT Gateway nếu có cấu hình sau |
| Private app subnet | Dùng cho Backend EC2 hoặc Worker |
| Private DB subnet | Dùng cho RDS PostgreSQL |

Việc chia subnet theo vai trò giúp hệ thống dễ quản lý hơn, đồng thời giảm rủi ro để backend hoặc database bị public ngoài ý muốn.



## Kết luận

Sau bước này, người thực hiện cần đảm bảo:

- Các subnet thuộc đúng VPC `socai-dev-vpc`.
- Chỉ sử dụng subnet có CIDR thuộc dải `10.20.0.0/16`.
- Public subnet dùng cho ALB hoặc NAT Gateway nếu có.
- Private app subnet dùng cho Backend EC2 hoặc Worker.
- Private DB subnet dùng cho RDS PostgreSQL.
- Các subnet đang ở trạng thái `Available`.
- Các subnet `172.31.x.x` thuộc default VPC không được dùng trong project.

Phần tiếp theo sẽ kiểm tra **Route Tables**.
