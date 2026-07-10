---
title : "Troubleshooting và Checklist"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.4.8. </b> "
---

## Troubleshooting

| Lỗi | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| CloudFront trả 403 | Thiếu OAC hoặc bucket policy | Kiểm tra OAC và S3 bucket policy |
| CloudFront trả 404 khi refresh route | Thiếu SPA fallback | Thêm custom error response 403/404 về `/index.html` |
| Dashboard trắng | Thiếu assets hoặc cache cũ | Upload lại `dist/` và invalidate `/*` |
| UI vẫn là bản cũ | CloudFront cache | Tạo invalidation mới |
| API call lỗi | Sai backend endpoint, CORS hoặc mock mode | Kiểm tra frontend env và backend readiness |
| WAF chặn nhầm demo traffic | Rule scope quá rộng | Kiểm tra sampled requests và chỉnh rule |

## Validation checklist

| Check | Expected result |
|---|---|
| Frontend build | Có `dist/` |
| S3 static bucket | Private bucket tồn tại |
| CloudFront | Distribution deployed |
| OAC | Bucket vẫn private |
| SPA fallback | Refresh route hoạt động |
| WAF | Web ACL gắn với CloudFront |
| Invalidation | Cache refresh completed |
| Browser validation | Static assets trả response thành công |

Không claim live telemetry, production API routing hoặc WebSocket readiness nếu chưa có backend evidence tương ứng.

