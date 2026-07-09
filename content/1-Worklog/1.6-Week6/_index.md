---
title: "Week 6 - S3 Static Website, CloudFront Delivery & Object Lifecycle"
date: 2026-05-29
weight: 6
chapter: false
draft: false
pre: " <b> 1.6. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary service | Amazon S3 |
| Supporting services | Amazon CloudFront, S3 Versioning, S3 Lifecycle, Bucket Policy |
| Main objective | Build an object-storage and static-delivery pattern for FCAJ project artifacts |
| Primary Region | `ap-southeast-1` - Asia Pacific (Singapore) |
| Storage model | Private raw objects plus selected public website files |
| Delivery model | CloudFront distribution using the S3 static website endpoint as origin |
| Data focus | Zeek-style raw dataset, static website files, versioning demo objects |
| Week status | Implemented with selected public evidence; time estimates require owner confirmation |

## Evidence Source Note

The Week 6 evidence was prepared from the existing S3 lab notes under:

```text
FCAJ-Internship/01_AWS-Labs/S3/
```

The saved screenshots were captured around `29/05/2026`. The original S3 and CloudFront resources may already have been deleted, so this page uses the saved lab notes and screenshots as evidence.

Raw screenshots are stored privately. The public report uses sanitized copies under:

```text
static/images/worklog/week6/
```

This lab used an S3 static website endpoint with a selective bucket policy and CloudFront. It did not implement private S3 delivery through CloudFront origin access control. Private-bucket CloudFront delivery remains a future hardening item.

## Objective

The objective of Week 6 was to understand how Amazon S3 can support the Hybrid IDS/AI SOC project as an object-storage layer and how CloudFront can serve selected static content through a CDN endpoint.

The lab focused on four connected capabilities:

- Store security-related objects such as raw Zeek-style datasets.
- Keep sensitive raw objects private by default.
- Publish only selected static website files.
- Use versioning and lifecycle rules to manage object history and cleanup.

## Project Context

The SOC project produces different types of data. Not all of them belong in the same storage service.

Amazon RDS is useful for structured alert records and dashboard queries, as tested in Week 5. Amazon S3 is a better fit for larger object-based data such as logs, CSV datasets, model files, static reports, and frontend build artifacts.

Week 6 therefore tested a storage and delivery pattern that can support both the learning lab and the future SOC dashboard:

```text
Raw Zeek-style dataset -> private S3 object under raw/zeek/
Static website files   -> selected public S3 objects
CloudFront             -> HTTPS viewer endpoint in front of the S3 website endpoint
Versioning             -> object rollback and accidental overwrite protection
Lifecycle rule         -> cleanup for old versions and delete markers
```

## S3 and CloudFront Architecture Overview

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-s3-cloudfront-architecture.svg" alt="Week 6 S3 and CloudFront architecture overview">
  <figcaption>Figure 1 - Week 6 storage and static-delivery architecture using S3, selective website access, CloudFront, versioning, and lifecycle cleanup.</figcaption>
</figure>

The architecture intentionally separates private project artifacts from static website files. Raw security data is uploaded to S3 but is not made public. Only `index.html` and `error.html` are exposed through the lab bucket policy so the S3 website endpoint and CloudFront distribution can serve the static site.

## Daily Worklog

| Activity | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| S3 bucket baseline | 29/05/2026 | Estimated 3 hours | Created the S3 lab bucket in `ap-southeast-1`, reviewed bucket naming, object ownership, Block Public Access, versioning default state, tags, and default SSE-S3 encryption | A private S3 bucket baseline was prepared for project artifacts | Keep the bucket private by default before publishing any website file | Upload project-style objects |
| Raw object upload and private access test | 30/05/2026 | Estimated 3.5 hours | Created the logical prefix `raw/zeek/`, uploaded `conn_dataset.csv`, and opened the object URL from the browser | The raw dataset object existed in S3 and direct browser access returned `AccessDenied` | Security logs and datasets should not be public website assets | Enable website hosting for selected files only |
| Static website hosting and bucket policy | 31/05/2026 | Estimated 4 hours | Created `index.html` and `error.html`, enabled S3 static website hosting, disabled bucket-level Block Public Access for the lab, and added a selective bucket policy | The S3 website endpoint served the static FCAJ page | The policy allowed only website files, not the entire bucket prefix | Add CloudFront in front of the website endpoint |
| CloudFront delivery validation | 01/06/2026 | Estimated 4 hours | Created a CloudFront distribution, configured the S3 static website endpoint as the origin, used the default CloudFront certificate, and tested the CloudFront domain | The website was reachable through a CloudFront URL | The origin used HTTP because S3 website endpoints do not support HTTPS to origin | Add versioning and lifecycle management |
| Versioning, lifecycle, and object organization | 02/06/2026 | Estimated 4 hours | Enabled S3 Versioning, uploaded multiple versions of a demo object, reviewed delete-marker behavior, created a lifecycle rule for `versioning-demo/`, and documented cleanup behavior | Object history and lifecycle cleanup were validated for the lab prefix | Lifecycle rules do not execute immediately; they run asynchronously based on age and conditions | Use Week 7 to move from cloud storage to local IDS telemetry architecture |

## Technical Implementation Summary

The Week 6 lab created an S3 bucket named `fcaj-s3-lab-thienit-20260529` in `ap-southeast-1`.

The bucket started as private. Object Ownership used ACLs disabled, default encryption used SSE-S3, and Block Public Access was enabled during the initial raw-object upload stage.

A Zeek-style dataset was uploaded under:

```text
raw/zeek/conn_dataset.csv
```

The direct object URL returned `AccessDenied`, which was the expected result for raw security data.

For the static website portion, the bucket hosted `index.html` and `error.html`. The bucket policy was deliberately scoped to those two website files instead of using a broad `bucket/*` policy. CloudFront was then configured with the S3 static website endpoint as the origin and served the website through the CloudFront distribution domain.

The lab also enabled S3 Versioning and created a lifecycle rule named `cleanup-versioning-demo` for the `versioning-demo/` prefix.

## S3 Storage Baseline

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e01-s3-bucket-created.png" alt="S3 bucket created">
  <figcaption>Figure 2 - The S3 lab bucket was created in `ap-southeast-1` and used as the object-storage baseline.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The bucket name and Region are visible, and the bucket was prepared before any website access was configured.  
**Project value:** S3 provides a durable object layer for raw logs, datasets, model artifacts, and report files.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e02-raw-zeek-object-uploaded.png" alt="Raw Zeek object uploaded to S3">
  <figcaption>Figure 3 - A Zeek-style dataset object was uploaded under the `raw/zeek/` logical prefix.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** `raw/zeek/conn_dataset.csv` is treated as an S3 object key, not as a real filesystem folder.  
**Project value:** The SOC project can store raw telemetry and generated datasets in object storage before later processing.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e03-private-object-access-denied.png" alt="Private S3 object access denied">
  <figcaption>Figure 4 - Direct browser access to the raw dataset object returned an AccessDenied response.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The raw object was not publicly readable through its direct object URL.  
**Security value:** Raw security logs and datasets should remain private unless a controlled application path explicitly authorizes access.

## Static Website Hosting and Bucket Policy

S3 static website hosting was enabled with:

```text
Index document: index.html
Error document: error.html
```

The lab intentionally made only the website files public. This is different from a broad bucket policy that exposes every object under the bucket.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e05-selective-bucket-policy.png" alt="Selective S3 bucket policy for website files">
  <figcaption>Figure 5 - The bucket policy grants public read access only to `index.html` and `error.html`.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The policy uses `s3:GetObject` for two specific resources rather than `arn:aws:s3:::bucket-name/*`.  
**Scope:** This is a lab-stage static website pattern. It is safer than making the whole bucket public, but it is not the same as private-bucket CloudFront delivery.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e04-static-website-success.png" alt="S3 static website endpoint success">
  <figcaption>Figure 6 - The S3 static website endpoint served the FCAJ static website successfully.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The browser rendered the `FCAJ - S3 Static Website` page through the S3 website endpoint.  
**Security note:** S3 website endpoints use HTTP. CloudFront is added next to provide an HTTPS viewer endpoint.

## CloudFront Delivery

CloudFront was configured in front of the S3 static website endpoint.

The important distinction is:

| Origin option | Week 6 status | Note |
| --- | --- | --- |
| S3 static website endpoint | Used | Required for S3 website hosting behavior such as index and error documents |
| S3 REST endpoint with private origin access | Not implemented | Future production hardening item |

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e06-cloudfront-origin-website-endpoint.png" alt="CloudFront origin uses S3 website endpoint">
  <figcaption>Figure 7 - CloudFront was configured with the S3 website endpoint as the origin domain.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The origin domain used `s3-website-ap-southeast-1.amazonaws.com`, not the S3 REST endpoint.  
**Scope:** This matches the lab evidence and avoids overstating the security model.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e07-cloudfront-distribution-detail.png" alt="CloudFront distribution detail">
  <figcaption>Figure 8 - The CloudFront distribution was created for the FCAJ S3 static website lab.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The distribution had a CloudFront domain name and `index.html` as the default root object.  
**Cost note:** The lab used the default CloudFront domain and did not add custom domain, ACM certificate, WAF, or paid edge-security features.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e08-cloudfront-url-success.png" alt="CloudFront URL served the static website">
  <figcaption>Figure 9 - The CloudFront URL served the FCAJ static website successfully.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The same static page was rendered through the CloudFront distribution domain.  
**Project value:** This pattern can later serve the SOC frontend or public internship report assets.

## Versioning and Lifecycle Management

S3 Versioning was used to understand object history, overwrites, and delete markers.

The lab used a small object under:

```text
versioning-demo/version-demo.txt
```

This avoided repeatedly uploading the larger dataset file just to test version behavior.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e09-version-history.png" alt="S3 object version history">
  <figcaption>Figure 10 - S3 Versioning showed multiple versions of the same object key.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** Uploading the same object key after versioning was enabled created multiple versions instead of simply overwriting the previous object.  
**Project value:** Versioning can protect project reports, model artifacts, and small configuration files from accidental overwrite.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e10-lifecycle-rule-enabled.png" alt="S3 lifecycle rule enabled">
  <figcaption>Figure 11 - The `cleanup-versioning-demo` lifecycle rule was enabled for version-cleanup testing.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The lifecycle rule was filtered and enabled for cleanup behavior rather than applied blindly to the whole bucket.  
**Cost value:** Versioning can increase storage usage, so lifecycle policies are needed to prevent old versions and delete markers from accumulating.

## Validation Matrix

| Test | Expected result | Actual result | Status |
| --- | --- | --- | --- |
| S3 bucket creation | Bucket exists in `ap-southeast-1` | Bucket `fcaj-s3-lab-thienit-20260529` was created | Pass |
| Raw object upload | Dataset object is stored under a project-style prefix | `raw/zeek/conn_dataset.csv` was uploaded | Pass |
| Raw object privacy | Direct browser access should not expose the dataset | Object URL returned AccessDenied | Pass |
| Static website files | `index.html` and `error.html` can be used as website files | Website endpoint rendered the FCAJ static page | Pass |
| Bucket policy scope | Website policy should not expose the whole bucket | Policy listed only `index.html` and `error.html` | Pass |
| CloudFront origin | Distribution points to S3 website endpoint | Origin domain used the website endpoint | Pass |
| CloudFront delivery | CloudFront URL renders the static website | CloudFront domain returned the FCAJ page | Pass |
| Versioning behavior | Re-uploading same key creates versions | Version list showed multiple object versions | Pass |
| Lifecycle management | Cleanup rule exists for old versions/delete markers | `cleanup-versioning-demo` rule was enabled | Pass |
| Private S3 origin through CloudFront | Bucket remains fully private while CloudFront alone serves content | Not implemented in this lab | Future improvement |

## Security, Cost and Cleanup Decisions

| Decision | Reason |
| --- | --- |
| Keep raw dataset objects private | Raw security logs and generated datasets should not be public web assets |
| Use a selective bucket policy | Only `index.html` and `error.html` needed public read access for the lab website |
| Avoid broad `bucket/*` public policy | Prevent accidental exposure of `raw/zeek/` objects |
| Use CloudFront for viewer delivery | Provides a cleaner public URL and HTTPS viewer access for the static website |
| Use the S3 website endpoint honestly | The evidence supports website endpoint origin, not private REST-origin hardening |
| Defer private-origin hardening | Requires new evidence and a different CloudFront/S3 access model |
| Use Versioning on small demo object | Avoid unnecessary duplicate storage for large datasets |
| Add lifecycle cleanup rule | Reduce long-term cost from noncurrent versions and delete markers |
| Avoid custom domain and WAF in the lab | Useful for production, unnecessary for short FCAJ service validation |
| Keep screenshots redacted | Prevent AWS account identifiers and request identifiers from entering the public report |

## Data Placement Decision

| Data type | Week 6 storage decision | Reason |
| --- | --- | --- |
| Raw Zeek/Suricata logs | S3 private objects | Large object files, not query rows |
| CSV datasets | S3 private objects | Good fit for batch processing and later AI pipelines |
| Model artifacts | S3 private objects | Object storage fits model files and versioned outputs |
| Static frontend/report files | S3 website files and CloudFront | Browser-facing static content |
| Final alert records | RDS PostgreSQL | Requires filtering, sorting, query, and dashboard persistence |

## Issues and Troubleshooting

| Issue | Resolution |
| --- | --- |
| Static website endpoint did not mean objects were automatically public | Uploaded `index.html` and `error.html`, then configured a bucket policy for those files |
| Direct object URL for raw dataset returned AccessDenied | Treated this as expected behavior because raw logs should stay private |
| A broad public bucket policy would expose raw dataset objects | Used a resource list for only `index.html` and `error.html` |
| S3 website endpoint uses HTTP | Added CloudFront for HTTPS viewer access through the CloudFront domain |
| CloudFront origin selection can be confusing | Used the S3 website endpoint because this lab depended on static website hosting behavior |
| Versioning can increase storage cost | Added a lifecycle rule for old versions and delete markers in the demo prefix |
| Lifecycle cleanup does not happen immediately | Documented that lifecycle processing is asynchronous and age-based |

## Lessons Learned

S3 is an object store, not a traditional folder filesystem. Prefixes such as `raw/zeek/` are part of the object key and help organize project artifacts logically.

Public access should be designed per use case. Raw telemetry and datasets should stay private, while selected website files can be made public for a static website lab.

S3 static website hosting and private CloudFront-origin delivery are different architecture patterns. The Week 6 lab used the website endpoint pattern, so the report should not claim private-bucket CloudFront access without separate evidence.

Versioning is useful but not free. Every noncurrent object version can add storage cost, so lifecycle rules are an important companion control.

## Weekly Outcome

By the end of Week 6, the project had a validated S3 and CloudFront storage-delivery pattern.

The lab showed how to upload raw SOC-style data to S3, keep that data private, publish selected website files, serve the static page through CloudFront, and use Versioning plus Lifecycle rules to manage object history and cleanup.

This creates a stronger storage foundation for future SOC datasets, frontend artifacts, reports, and model files.

## Next Week Plan

Week 7 will move away from AWS managed services and focus on the local IDS lab architecture that generates realistic security telemetry for the project.

The focus will be:

- pfSense network segmentation
- Suricata and Zeek telemetry sources
- Kali attack traffic generation
- Victim Ubuntu workload
- ELK or log-review workflow
- Traffic-flow diagrams and lab evidence
