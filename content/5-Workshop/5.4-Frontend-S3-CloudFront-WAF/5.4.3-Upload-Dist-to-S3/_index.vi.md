---
title : "Upload Dist to S3"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

## Mục tiêu

Upload nội dung bên trong `dist/` lên static bucket. File `index.html` phải nằm ở root bucket, không nằm dưới folder con `dist/`.

Đường dẫn console:

```text
S3 -> Buckets -> static frontend bucket -> Objects -> Upload
```

Tùy chọn CLI:

```bash
aws s3 sync dist/ s3://<S3_STATIC_BUCKET>/ --delete --region <AWS_REGION>
```

Màn hình upload ghi lại thao tác đưa build artifact lên S3.

![S3 upload console](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-10-s3-static-upload-console.png)

Object view ở root xác nhận `index.html` và `assets/` nằm đúng vị trí CloudFront cần.

![S3 objects root](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-11-s3-static-objects-root.png)

Object `index.html` là entrypoint của frontend khi truy cập root path.

![S3 index object](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-12-s3-static-index-object.png)

Folder assets chứa JavaScript và CSS bundles do Vite tạo ra.

![S3 assets folder](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-13-s3-static-assets-folder.png)

CLI sync output là evidence optional nếu nhóm deploy bằng terminal thay vì console.

![S3 sync CLI result](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-14-s3-sync-cli-result.png)

