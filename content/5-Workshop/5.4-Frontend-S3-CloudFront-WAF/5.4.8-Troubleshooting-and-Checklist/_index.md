---
title : "Troubleshooting and Checklist"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.4.8. </b> "
---

## Troubleshooting

| Issue | Common cause | Fix |
|---|---|---|
| CloudFront returns 403 | OAC or bucket policy is missing | Check OAC and S3 bucket policy |
| CloudFront returns 404 on frontend route | SPA fallback is missing | Add 403/404 custom error responses to `/index.html` |
| Dashboard is blank | Assets missing or stale cache | Re-upload `dist/` and invalidate `/*` |
| UI still shows old version | CloudFront cache | Create a new invalidation |
| API calls fail | Backend endpoint, CORS, or mock mode mismatch | Verify frontend env and backend readiness |
| WAF blocks expected demo traffic | Rule scope too broad | Review sampled requests and adjust rules |

## Validation checklist

| Check | Expected result |
|---|---|
| Frontend build | `dist/` exists |
| S3 static bucket | Private bucket exists |
| CloudFront | Distribution is deployed |
| OAC | Bucket can stay private |
| SPA fallback | Refreshing routes works |
| WAF | Web ACL is associated with CloudFront |
| Invalidation | Cache refresh is completed |
| Browser validation | Static assets return successful responses |

Do not claim live telemetry, production API routing, or WebSocket readiness unless matching backend evidence exists.

