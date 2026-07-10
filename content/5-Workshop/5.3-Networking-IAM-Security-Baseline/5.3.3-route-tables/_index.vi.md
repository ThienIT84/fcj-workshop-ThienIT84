---
title: "Route Tables"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

## Giới thiệu

Route table quyết định traffic trong subnet sẽ đi đâu. Trong hệ thống **Hybrid SOC trên AWS**, route table được dùng để tách đường đi của public subnet, private app subnet và private DB subnet.

Sau khi đã tạo VPC và subnets, cần kiểm tra các route table thuộc VPC `socai-dev-vpc`.


## Bước 1: Vào trang Route Tables

Trên AWS Console, mở dịch vụ **VPC**, sau đó chọn **Route tables** ở menu bên trái.

Chỉ kiểm tra các route table thuộc VPC của project:

```text
VPC: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
```

Hình dưới đây cho thấy danh sách route table trong VPC của project.

![Route Tables List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-03-01-route-tables-list.jpg)


## Bước 2: Kiểm tra các Route Tables hiện có

Trong project hiện tại, các route table chính gồm:

| Route table name | Vai trò | Ghi chú |
|---|---|---|
| `socai-dev-rtb-public` | Public route table | Được associate với 2 public subnets |
| `socai-dev-rt-private-a` | Private app route table AZ A | Được associate với private subnet A |
| `social-dev-rt-private-b` | Private app route table AZ B | Được associate với private subnet B |
| `socai-dev-rt-db` | Database route table | Được associate với 2 private DB subnets |

Ngoài ra còn có route table mặc định của VPC. Route table này không phải route table chính dùng để mô tả các lớp public, private app và database trong workshop.


## Bước 3: Kiểm tra subnet association

Ở cột **Explicit subnet associations**, cần kiểm tra route table đang được gắn với đúng subnet hay chưa.

Theo hình hiện tại:

```text
socai-dev-rtb-public -> 2 subnets
socai-dev-rt-db -> 2 subnets
socai-dev-rt-private-a -> private subnet A
social-dev-rt-private-b -> private subnet B
```

Việc associate đúng subnet giúp đảm bảo:

- Public subnet dùng đúng public route table.
- Private app subnet dùng đúng private route table.
- Private DB subnet dùng đúng database route table.
- Database subnet không bị gắn nhầm với public route table.


## Bước 4: Kiểm tra trạng thái route table

Cần đảm bảo các route table thuộc đúng VPC:

```text
vpc-04c78e55ae4ceec8a
```

Không sử dụng route table thuộc default VPC:

```text
vpc-0bc9182791a842901
```

Route table của default VPC không thuộc project Hybrid SOC nên không dùng trong workshop này.

## Vai trò Route Table trong workshop

| Loại route table | Vai trò |
|---|---|
| Public route table | Dùng cho public subnet, thường route ra Internet Gateway |
| Private app route table | Dùng cho Backend EC2 hoặc Worker trong private subnet |
| DB route table | Dùng cho private DB subnet của RDS PostgreSQL |

## Kết luận

Sau bước này, người thực hiện cần đảm bảo:

- Route tables thuộc đúng VPC `socai-dev-vpc`.
- Public route table được associate với public subnets.
- Private app route tables được associate với private app subnets.
- DB route table được associate với private DB subnets.
- Route table thuộc default VPC không được dùng cho project này.

Phần tiếp theo sẽ kiểm tra **Internet Gateway**.
