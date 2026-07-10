---
title : "Frontend S3, CloudFront and WAF"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

# Frontend S3, CloudFront and WAF

## Objective

This section deploys the Hybrid SOC dashboard frontend as static assets on Amazon S3, serves it through CloudFront, and protects the public entrypoint with AWS WAF.

```text
React/Vite source
-> dist/
-> S3 Static Frontend Bucket
-> CloudFront Distribution with OAC
-> WAF Web ACL
-> User browser
```

The S3 Static Frontend Bucket is only for `index.html` and frontend assets. It is different from the S3 Data Bucket used by the pipeline for raw logs, reports, and evidence. The bucket remains private; users access the dashboard through CloudFront.

During the current workshop phase, the dashboard may use mock or replay data while the backend API, SQS, S3 Data Bucket, RDS, and WebSocket flow are being completed. Screenshots that show mock data are treated as UI and workflow validation, not live telemetry.

## Subsections

| Section | Purpose |
|---|---|
| 5.4.1 Build Frontend | Build the React/Vite dashboard and verify `dist/` |
| 5.4.2 S3 Static Frontend Bucket | Prepare a private bucket for frontend assets |
| 5.4.3 Upload Dist to S3 | Upload `index.html` and `assets/` correctly |
| 5.4.4 CloudFront OAC and Bucket Policy | Serve private S3 content through CloudFront |
| 5.4.5 SPA Fallback and Cache Invalidation | Support frontend routes and refresh cache |
| 5.4.6 WAF Web ACL | Attach edge protection to the dashboard entrypoint |
| 5.4.7 Dashboard Access and Validation | Validate CloudFront, dashboard, and browser evidence |
| 5.4.8 Troubleshooting and Checklist | Summarize common issues and final checks |

