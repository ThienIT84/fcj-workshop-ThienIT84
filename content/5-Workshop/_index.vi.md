---
title: "Workshop FCAJ Hybrid SOC/AWS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


## Tổng quan

Workshop này mô tả project FCAJ Hybrid SOC/AWS: một pipeline SOC hybrid bắt đầu từ security evidence ở local lab, đưa event lên AWS, xử lý qua các cloud data service và AI/Fusion component, rồi chuẩn bị final alert cho lưu trữ, monitoring và dashboard downstream.

Workshop hiện được tổ chức thành 11 section chính, đã được ghi nhận đầy đủ với thông tin cấu hình và ảnh chụp màn hình xác thực thực tế.

## Nội dung workshop

1. [Overview and Architecture](5.1-Overview-and-Architecture/)
2. [Prerequisites and Naming](5.2-Prerequisites-and-Naming/)
3. [Networking, IAM and Security Baseline](5.3-Networking-IAM-Security-Baseline/)
4. [Frontend S3, CloudFront and WAF](5.4-Frontend-S3-CloudFront-WAF/)
5. [Backend EC2 Runtime](5.5-Backend-EC2-Runtime/)
6. [SQS, DLQ and S3 Data Bucket](5.6-SQS-DLQ-and-S3-Data-Bucket/)
7. [RDS, Secrets and Worker Pipeline](5.7-RDS-Secrets-Worker-Pipeline/)
8. [AWS Pipeline Validation and Dashboard Evidence](5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/)
9. [AI Fusion and Dashboard Validation](5.9-AI-Fusion-and-Dashboard-Validation/)
10. [Monitoring, Audit and Notification](5.10-Monitoring-Audit-Notification/)
11. [Cleanup and Cost Control](5.11-Cleanup/)

## Nguyên tắc evidence

Chỉ đưa evidence đã xác minh và đã redact vào các trang workshop. Tất cả cấu hình được ghi nhận đều đi kèm tài liệu triển khai và xác thực thực tế trên hệ thống.
