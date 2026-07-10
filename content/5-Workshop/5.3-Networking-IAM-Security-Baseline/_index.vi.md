---
title: "Mạng, IAM và Nền tảng An ninh"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---


## Giới thiệu

Phần này kiểm tra nền tảng **network, IAM và security baseline** để EC2 Backend có thể chạy an toàn và kết nối đến các service cần thiết như SQS, S3, RDS, Secrets Manager và CloudWatch.

Các nội dung cần kiểm tra gồm:

- VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- IAM Role for EC2 Backend

Nếu service nào chưa cấu hình, cần ghi rõ là **chưa cấu hình** hoặc **phase mở rộng**, không claim là đã triển khai.

## Mục tiêu

Sau phần này, người thực hiện cần xác định được:

- VPC đang dùng cho project.
- Subnet nào dùng cho public, app và database layer.
- Route table nào cho public/private subnet.
- NAT Gateway đã có hay chưa.
- Security Group backend đang mở port nào.
- EC2 Backend có dùng IAM Role thay vì access key hay không.

## Danh sách bài trong section này

- [VPC Overview](./5.3.1-vpc-overview/)
- [Subnets](./5.3.2-subnets/)
- [Route Tables](./5.3.3-route-tables/)
- [Internet Gateway](./5.3.4-internet-gateway/)
- [NAT Gateway](./5.3.5-nat-gateway/)
- [Security Groups](./5.3.6-security-groups/)
- [IAM Role for EC2 Backend](./5.3.7-iam-role-backend/)
