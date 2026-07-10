---
title : "WAF Web ACL"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.4.6. </b> "
---

## Objective

Attach AWS WAF to the CloudFront entrypoint. WAF protects the dashboard edge layer, but the rule scope must not accidentally block security evidence that the SOC system needs to ingest.

The create Web ACL screen records the initial WAF setup for the CloudFront scope.

![Create WAF Web ACL](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-30-waf-create-web-acl.png)

The associated resource evidence confirms that the Web ACL is attached to the CloudFront distribution.

![WAF associated CloudFront resource](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-31-waf-associated-cloudfront.png)

The rules list shows the protections enabled for the edge entrypoint.

![WAF rules list](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-32-waf-rules-list.png)

The rule scope note is important for this project: evidence such as SQL injection or XSS payloads may be valid SOC input, so WAF should protect the dashboard without invalidating the ingestion demo.

![WAF rule scope note](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-33-waf-rule-scope-note.png)

If WAF logging is not fully configured, keep it as a  instead of claiming centralized WAF logging.

![WAF metrics and logging](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-34-waf-metrics-logging-placeholder.png)
