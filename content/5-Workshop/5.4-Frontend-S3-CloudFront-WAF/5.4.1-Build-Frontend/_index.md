---
title : "Build Frontend"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

## Objective

Build the React/Vite dashboard and confirm that the deployment artifact is ready for S3.

```bash
cd frontend
pnpm install
pnpm build
```

Expected output:

```text
dist/
├── index.html
└── assets/
```

Before building, review the production environment file and redact sensitive values in screenshots. Mock mode is acceptable while the real backend endpoint is not ready.

The source folder evidence shows the dashboard project and build inputs used for this workshop.

![Frontend source folder](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-01-frontend-folder.png)

The environment screenshot explains how the frontend receives API and WebSocket configuration. Sensitive endpoint or token values must remain .

![Frontend environment ](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-02-frontend-env.png)

The build output confirms that the frontend compiles successfully.

![Frontend build success](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-03-frontend-build-success.png)

The `dist/` folder is the artifact that will be uploaded to the S3 Static Frontend Bucket.

![Frontend dist folder](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-04-frontend-dist-folder.png)

