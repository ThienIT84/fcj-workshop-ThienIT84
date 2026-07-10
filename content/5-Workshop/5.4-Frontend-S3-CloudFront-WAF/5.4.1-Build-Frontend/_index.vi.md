---
title : "Build Frontend"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

## Mục tiêu

Build React/Vite dashboard và xác nhận artifact đã sẵn sàng để upload lên S3.

```bash
cd frontend
pnpm install
pnpm build
```

Kết quả mong đợi:

```text
dist/
├── index.html
└── assets/
```

Trước khi build, kiểm tra file môi trường production và redact giá trị nhạy cảm khi chụp screenshot. Mock mode có thể dùng trong lúc endpoint backend thật chưa sẵn sàng.

Ảnh source folder cho thấy project dashboard và input dùng để build trong workshop.

![Frontend source folder](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-01-frontend-folder.png)

Ảnh môi trường giải thích frontend nhận API và WebSocket configuration như thế nào. Endpoint hoặc token nhạy cảm phải được redact.

![Frontend environment ](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-02-frontend-env.png)

Build output xác nhận frontend compile thành công.

![Frontend build success](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-03-frontend-build-success.png)

Thư mục `dist/` là artifact sẽ được upload lên S3 Static Frontend Bucket.

![Frontend dist folder](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-04-frontend-dist-folder.png)

