---
title : "S3 Data Bucket và prefixes"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---


## Mục tiêu

Xác minh S3 Data Bucket và ghi lại prefix convention dùng cho raw evidence, normalized event, report và export.

## Bước 1: Xác minh data bucket

Mở Amazon S3 và chọn `socai-dev-data-************`. Bucket này dùng cho SOC data, không phải static website hosting.

![S3 data bucket overview](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-05-s3-data-bucket-overview.png)

## Bước 2: Kiểm tra top-level prefixes

Dùng prefix ổn định để backend, worker và manual validation có thể tìm object nhất quán:

```text
raw/
normalized/
reports/
exports/
```

![S3 prefix structure](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-07-s3-prefix-structure.png)

## Bước 3: Kiểm tra Zeek prefixes

Zeek evidence nên được tách theo log type. Các ví dụ mong đợi:

```text
raw/zeek/http/
raw/zeek/conn/
```

![S3 Zeek prefixes](/images/5-Workshop/5.6-SQS-DLQ-and-S3-Data-Bucket/w-tin-08-s3-zeek-prefixes.png)
