---
title : "CloudFront OAC và Bucket Policy"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

## Mục tiêu

Tạo hoặc kiểm tra CloudFront distribution phục vụ private S3 frontend bucket thông qua Origin Access Control.

Cấu hình chính:

| Nhóm | Giá trị |
|---|---|
| Origin | S3 Static Frontend Bucket |
| Origin access | Origin Access Control |
| Viewer protocol policy | Redirect HTTP to HTTPS |
| Allowed methods | GET, HEAD |
| Default root object | `index.html` |

Origin configuration cho thấy CloudFront trỏ tới static frontend bucket, không phải data bucket.

![CloudFront create origin](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-15-cloudfront-create-origin.png)

OAC cho phép CloudFront đọc private S3 objects mà không mở bucket ra Internet.

![CloudFront OAC configuration](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-16-cloudfront-oac-config.png)

Default behavior được cấu hình cho static content và redirect viewer sang HTTPS.

![CloudFront default behavior](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-17-cloudfront-default-behavior.png)

Default root object giúp `/` trả về dashboard entrypoint.

![CloudFront default root object](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-18-cloudfront-default-root-object.png)

Evidence distribution deployed xác nhận CloudFront đã sẵn sàng phục vụ frontend.

![CloudFront distribution deployed](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-19-cloudfront-distribution-deployed.png)

CloudFront có thể sinh bucket policy gợi ý cho OAC. Bucket vẫn private và chỉ cho distribution source ARN truy cập.

![CloudFront OAC policy prompt](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-20-cloudfront-oac-policy-prompt.png)

Bucket policy evidence cho thấy CloudFront service access mà không bật public read.

![S3 bucket policy for OAC](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-21-s3-bucket-policy-oac.png)

Trang permissions xác nhận Block Public Access vẫn bật sau khi áp dụng OAC policy.

![S3 permissions after OAC policy](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-22-s3-permissions-after-policy.png)

