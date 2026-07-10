---
title : "Dashboard Access and Validation"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.4.7. </b> "
---

## Objective

Validate that users can access the dashboard through CloudFront and that browser-side evidence supports the deployment.

The dashboard home screenshot proves that CloudFront can serve the frontend application.

![Dashboard CloudFront home](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-38-dashboard-cloudfront-home.png)

The HTTP response check confirms that the CloudFront URL returns a successful response.  identifiers in headers are not required for the workshop.

![Curl CloudFront 200](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-43-curl-cloudfront-200.png)

DevTools static asset evidence confirms that JavaScript and CSS bundles are loading successfully.

![DevTools static assets 200](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-57-devtools-static-assets-200.png)

The API call evidence is used only to show frontend data-source behavior. If it is mock or replay mode, do not claim live backend telemetry.

![DevTools API call mock or real](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-58-devtools-api-call-mock-or-real.png)

The console screenshot is useful to confirm there are no critical browser errors after deployment.

![DevTools console warnings](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-60-devtools-console-warnings.png)

The API and WebSocket CloudFront behavior screenshots are future evidence. Do not claim these paths are production-ready unless the backend and ALB evidence also exist.

![API behavior ](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-26-api-behavior.png)

