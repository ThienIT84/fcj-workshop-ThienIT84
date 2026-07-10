---
title : "WAF Web ACL"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.4.6. </b> "
---

## Mục tiêu

Gắn AWS WAF vào CloudFront entrypoint. WAF bảo vệ edge layer của dashboard, nhưng rule scope không được chặn nhầm security evidence mà SOC system cần ingest.

Màn hình create Web ACL ghi lại cấu hình WAF ban đầu cho CloudFront scope.

![Create WAF Web ACL](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-30-waf-create-web-acl.png)

Associated resource evidence xác nhận Web ACL đã gắn với CloudFront distribution.

![WAF associated CloudFront resource](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-31-waf-associated-cloudfront.png)

Rules list cho thấy các protection đang bật cho edge entrypoint.

![WAF rules list](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-32-waf-rules-list.png)

Rule scope note rất quan trọng với project này: payload SQL injection hoặc XSS có thể là SOC input hợp lệ, nên WAF cần bảo vệ dashboard mà không phá demo ingestion.

![WAF rule scope note](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-33-waf-rule-scope-note.png)

Nếu WAF logging chưa cấu hình đầy đủ, giữ wording là  thay vì claim centralized WAF logging.

![WAF metrics and logging ](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-34-waf-metrics-logging.png)

