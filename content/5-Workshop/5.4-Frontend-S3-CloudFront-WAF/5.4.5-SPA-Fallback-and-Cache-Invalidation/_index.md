---
title : "SPA Fallback and Cache Invalidation"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

## Objective

Configure CloudFront so direct refreshes of frontend routes return `index.html`, then invalidate cached objects after deployment.

For a single-page application, configure custom error responses:

| Error | Response page | Response code |
|---|---|---|
| 403 | `/index.html` | 200 |
| 404 | `/index.html` | 200 |

The 403 custom error response helps route refreshes work when S3 denies direct object lookup for frontend paths.

![CloudFront 403 SPA fallback](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-23-cloudfront-error-page-403.png)

The 404 custom error response handles frontend routes that do not exist as physical S3 objects.

![CloudFront 404 SPA fallback](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-24-cloudfront-error-page-404.png)

This browser evidence confirms that refreshing a dashboard route still loads the application.

![Dashboard refresh route test](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-25-dashboard-refresh-route-test.png)

After uploading a new build, create an invalidation for `/*` so users receive the latest files.

![Create CloudFront invalidation](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-35-cloudfront-create-invalidation.png)

The completed invalidation proves that CloudFront cache has been refreshed for the deployment.

![CloudFront invalidation completed](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-36-cloudfront-invalidation-completed.png)

The dashboard evidence after invalidation confirms that the latest build is visible through CloudFront.

![Dashboard after invalidation](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-37-dashboard-after-invalidation.png)

