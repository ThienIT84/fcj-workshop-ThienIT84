---
title : "Frontend S3, CloudFront và WAF"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

# Frontend S3, CloudFront và WAF

## Mục tiêu

Phần này triển khai dashboard frontend của hệ thống Hybrid SOC dưới dạng static asset trên Amazon S3, phân phối qua CloudFront, và bảo vệ entrypoint bằng AWS WAF.

```text
React/Vite source
-> dist/
-> S3 Static Frontend Bucket
-> CloudFront Distribution với OAC
-> WAF Web ACL
-> User browser
```

S3 Static Frontend Bucket chỉ dùng cho `index.html` và frontend assets. Bucket này khác với S3 Data Bucket dùng trong pipeline để lưu raw log, report và evidence. Bucket vẫn private; người dùng truy cập dashboard thông qua CloudFront.

Ở giai đoạn workshop hiện tại, dashboard có thể dùng mock hoặc replay data trong lúc backend API, SQS, S3 Data Bucket, RDS và WebSocket flow đang được hoàn thiện. Các screenshot mock data được xem là evidence cho UI và workflow, không claim live telemetry.

## Các subsection

| Section | Mục đích |
|---|---|
| 5.4.1 Build Frontend | Build React/Vite dashboard và kiểm tra `dist/` |
| 5.4.2 S3 Static Frontend Bucket | Chuẩn bị private bucket cho frontend assets |
| 5.4.3 Upload Dist to S3 | Upload đúng `index.html` và `assets/` |
| 5.4.4 CloudFront OAC and Bucket Policy | Phục vụ private S3 content qua CloudFront |
| 5.4.5 SPA Fallback and Cache Invalidation | Hỗ trợ frontend route và refresh cache |
| 5.4.6 WAF Web ACL | Gắn edge protection cho dashboard entrypoint |
| 5.4.7 Dashboard Access and Validation | Validate CloudFront, dashboard và browser evidence |
| 5.4.8 Troubleshooting and Checklist | Tổng hợp lỗi thường gặp và checklist cuối |

