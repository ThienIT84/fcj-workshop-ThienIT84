---
title: "Vai trò IAM cho EC2 Backend"
date: 2026-07-08
weight: 7
chapter: false
pre: " <b> 5.3.7. </b> "
---

## Giới thiệu

IAM Role được dùng để cấp quyền cho **EC2 Backend** truy cập các AWS services như S3, SQS, Secrets Manager và CloudWatch.

Trong hệ thống **Hybrid SOC trên AWS**, Backend EC2 không nên sử dụng AWS access key hoặc secret key hard-code trong source code hoặc file `.env`. Thay vào đó, backend nên dùng IAM Role được gắn trực tiếp vào EC2 instance.

## Bước 1: Vào trang IAM Roles

Trên AWS Console, mở dịch vụ **IAM**, sau đó chọn **Roles**.

Tìm role backend của project:
```text
socai-backend-role
```

Nếu role thực tế trên AWS đang dùng tên khác, cần ghi đúng theo tên thật trong console.

![IAM Backend Role Summary](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-07-01-iam-backend-role-summary.jpg)

## Bước 2: Kiểm tra Role Summary

Trong trang role summary, cần kiểm tra các thông tin chính:

```text
Role name: socai-backend-role
Trusted entity: EC2
Use case: EC2 instance
```

Role này sẽ được gắn vào Backend EC2 để backend có thể gọi AWS services mà không cần access key.

Các quyền backend cần dùng gồm:
```text
S3
SQS
Secrets Manager
CloudWatch Logs
KMS nếu secret hoặc bucket dùng custom KMS key
```

## Bước 3: Kiểm tra Permissions

Mở tab **Permissions** để kiểm tra policy được gắn vào role.

IAM Role của backend cần có quyền tối thiểu để:
- Gửi message vào SQS.
- Ghi raw evidence vào S3 Data Bucket.
- Đọc secret kết nối RDS nếu backend cần dùng Secrets Manager.
- Ghi log lên CloudWatch nếu có cấu hình logging.
- Giải mã KMS nếu secret hoặc S3 object dùng KMS custom key.

Các permission cần kiểm tra:
```text
sqs:SendMessage
s3:PutObject
s3:GetObject
secretsmanager:GetSecretValue
logs:CreateLogStream
logs:PutLogEvents
kms:Decrypt
```

![IAM Backend Permissions](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-07-02-iam-backend-permissions.jpg)

## Bước 4: Kiểm tra Trust Relationship

Mở tab **Trust relationships** để kiểm tra service nào được phép assume role này.

Trust policy cần có principal là EC2:
```text
ec2.amazonaws.com
```

Nếu trust relationship không đúng, EC2 instance sẽ không thể sử dụng role này.

Ví dụ trust policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

![IAM Trust Policy EC2](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-07-03-iam-trust-policy-ec2.jpg)

## Bước 5: Kiểm tra role đã gắn vào EC2 Backend

Sau khi kiểm tra IAM Role, cần quay lại EC2 instance `socai-dev-backend-01` để xác nhận role đã được gắn vào instance.

Cách kiểm tra:
```text
EC2
-> Instances
-> Chọn socai-dev-backend-01
-> Security tab
-> IAM Role
```

IAM Role phải là role backend của project, ví dụ:
```text
socai-backend-role
```

Nếu EC2 chưa được gắn IAM Role, backend có thể gặp lỗi:
```text
Unable to locate credentials
AccessDenied
```
khi gọi S3, SQS hoặc Secrets Manager.

## Vai trò IAM Role trong workshop

| Thành phần | Vai trò |
|---|---|
| EC2 Backend | Chạy FastAPI service |
| IAM Role | Cấp quyền cho EC2 gọi AWS services |
| S3 | Lưu raw evidence |
| SQS | Nhận event message từ backend |
| Secrets Manager | Lưu secret kết nối database |
| CloudWatch | Ghi log và theo dõi runtime |
| KMS | Giải mã secret hoặc object nếu có cấu hình |

IAM Role giúp backend gọi AWS services an toàn hơn, không cần lưu credential trực tiếp trên server.

## Lưu ý

Không đưa các thông tin sau vào screenshot hoặc markdown:
- AWS access key.
- AWS secret key.
- Secret value.
- DB password.
- Private key `.pem`.
- Token hoặc API key.
- Nội dung `.env` chưa che.

Nếu policy có resource ARN chứa account ID, có thể crop hoặc che account ID nếu không cần thiết.

Trong môi trường production, IAM Role nên được cấp quyền theo nguyên tắc **least privilege**, chỉ cấp đúng action và đúng resource backend cần dùng.

## Kết luận

Sau bước này, người thực hiện cần đảm bảo:

- EC2 Backend sử dụng IAM Role thay vì access key.
- Role backend có trusted entity là EC2.
- Role có quyền cần thiết để gọi S3, SQS, Secrets Manager và CloudWatch.
- Nếu dùng KMS custom key, role cần có quyền `kms:Decrypt`.
- IAM Role đã được gắn vào EC2 Backend.
- Không có credential nhạy cảm bị hard-code trong source code hoặc `.env`.

Sau khi hoàn thành phần này, baseline network và security cho Backend EC2 đã sẵn sàng để tiếp tục kiểm tra backend runtime và pipeline xử lý event.

Phần tiếp theo sẽ kiểm tra **Backend EC2 Runtime**.
