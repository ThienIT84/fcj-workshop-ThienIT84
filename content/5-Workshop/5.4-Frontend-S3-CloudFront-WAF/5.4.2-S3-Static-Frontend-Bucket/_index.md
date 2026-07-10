---
title : "S3 Static Frontend Bucket"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

## Objective

Create or verify the private S3 bucket that stores frontend build artifacts. This bucket is not the data bucket used for SOC evidence.

Recommended settings:

| Setting | Value |
|---|---|
| Object ownership | ACLs disabled |
| Block Public Access | On |
| Default encryption | SSE-S3 or SSE-KMS |
| Purpose tag | StaticFrontend |

The create bucket screen records the bucket configuration used for the frontend static assets.

![Create S3 static bucket](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-05-s3-static-create-bucket.png)

The bucket overview confirms that the static frontend bucket exists in the selected AWS Region.

![S3 static bucket overview](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-06-s3-static-bucket-overview.png)

Block Public Access stays enabled because CloudFront reads the bucket through OAC instead of public S3 website hosting.

![S3 block public access](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-07-s3-static-block-public-access.png)

Default encryption protects the stored frontend objects at rest.

![S3 default encryption](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-08-s3-static-encryption.png)

Tags help separate frontend delivery resources from backend data pipeline resources.

![S3 static bucket tags](/images/5-Workshop/5.4-Frontend-S3-CloudFront-WAF/w-phu-09-s3-static-tags.png)

