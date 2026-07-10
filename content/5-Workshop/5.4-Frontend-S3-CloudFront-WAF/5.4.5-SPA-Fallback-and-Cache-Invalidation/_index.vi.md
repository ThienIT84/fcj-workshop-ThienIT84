---
title : "SPA Fallback và Cache Invalidation"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

## Mục tiêu

Cấu hình CloudFront để refresh trực tiếp frontend route vẫn trả về `index.html`, sau đó invalidate cache sau mỗi lần deploy.

Với single-page application, cấu hình custom error response:

| Error | Response page | Response code |
|---|---|---|
| 403 | `/index.html` | 200 |
| 404 | `/index.html` | 200 |

Custom error response 403 giúp route refresh hoạt động khi S3 từ chối lookup trực tiếp frontend path.

![CloudFront 403 SPA fallback](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-23-cloudfront-error-page-403.png)

Custom error response 404 xử lý các frontend route không tồn tại dưới dạng object vật lý trong S3.

![CloudFront 404 SPA fallback](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-24-cloudfront-error-page-404.png)

Browser evidence xác nhận refresh trực tiếp dashboard route vẫn load được ứng dụng.

![Dashboard refresh route test](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-25-dashboard-refresh-route-test.png)

Sau khi upload build mới, tạo invalidation cho `/*` để người dùng nhận file mới nhất.

![Create CloudFront invalidation](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-35-cloudfront-create-invalidation.png)

Invalidation completed chứng minh CloudFront cache đã được refresh cho lần deploy.

![CloudFront invalidation completed](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-36-cloudfront-invalidation-completed.png)

Dashboard evidence sau invalidation xác nhận build mới đã hiển thị qua CloudFront.

![Dashboard after invalidation](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-37-dashboard-after-invalidation.png)

