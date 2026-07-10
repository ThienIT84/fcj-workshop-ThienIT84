---
title : "S3 Static Frontend Bucket"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

## Mục tiêu

Tạo hoặc kiểm tra private S3 bucket dùng để lưu frontend build artifact. Bucket này không phải data bucket dùng cho SOC evidence.

Cấu hình khuyến nghị:

| Cấu hình | Giá trị |
|---|---|
| Object ownership | ACLs disabled |
| Block Public Access | On |
| Default encryption | SSE-S3 hoặc SSE-KMS |
| Purpose tag | StaticFrontend |

Màn hình create bucket ghi lại cấu hình bucket dùng cho frontend static assets.

![Create S3 static bucket](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-05-s3-static-create-bucket.png)

Bucket overview xác nhận static frontend bucket đã tồn tại trong AWS Region đã chọn.

![S3 static bucket overview](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-06-s3-static-bucket-overview.png)

Block Public Access vẫn bật vì CloudFront đọc bucket qua OAC, không dùng public S3 website hosting.

![S3 block public access](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-07-s3-static-block-public-access.png)

Default encryption bảo vệ frontend objects khi lưu trữ.

![S3 default encryption](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-08-s3-static-encryption.png)

Tags giúp phân biệt frontend delivery resources với backend data pipeline resources.

![S3 static bucket tags](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-09-s3-static-tags.png)

