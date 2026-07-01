# Week 6 Draft - S3 Static Website, CloudFront Delivery and Object Lifecycle

This file is the internal source and implementation guide for the Week 6 Hugo worklog page.

Target Hugo page:

```text
content/1-Worklog/1.6-Week6/_index.md
```

Public evidence folder:

```text
static/images/worklog/week6/
```

Original screenshots copied from the local FCAJ notes are stored privately under:

```text
private-evidence/week6/originals/
```

Do not embed raw screenshots directly from the Windows/WSL vault path.

## Positioning

Week 6 is:

```text
Week 6 - S3 Static Website, CloudFront Delivery & Object Lifecycle
```

This week focuses on:

- Amazon S3 bucket baseline
- Object key and prefix organization
- Private raw dataset object
- Static website hosting
- Selective bucket policy
- CloudFront distribution using the S3 website endpoint as origin
- S3 Versioning
- S3 Lifecycle cleanup rule

## Architecture Honesty Rule

The available evidence supports this pattern:

```text
S3 static website endpoint + selective bucket policy + CloudFront
```

The available evidence does not support this claim:

```text
Private S3 bucket with CloudFront-only access
```

Private S3 delivery through CloudFront origin access control should stay as a future improvement unless new evidence is collected.

## Source Material

Main source folder:

```text
/mnt/d/StorgeVault/Thiện học AI/FCAJ-Internship/01_AWS-Labs/S3/
```

Source images are stored in the Obsidian vault root:

```text
/mnt/d/StorgeVault/Thiện học AI/
```

## Daily Time Model

The public draft spreads the S3 and CloudFront lab across one work week:

| Date | Time estimate | Focus |
| --- | --- | --- |
| 29/05/2026 | Estimated 3 hours | S3 bucket baseline |
| 30/05/2026 | Estimated 3.5 hours | Raw object upload and AccessDenied validation |
| 31/05/2026 | Estimated 4 hours | Static website hosting and selective bucket policy |
| 01/06/2026 | Estimated 4 hours | CloudFront distribution and URL validation |
| 02/06/2026 | Estimated 4 hours | Versioning, delete marker review, lifecycle rule, object organization |

Total draft estimate:

```text
18.5 hours
```

These values should remain draft estimates until the project owner confirms or adjusts them.

## Public Evidence Map

| Public image | Source image | Use |
| --- | --- | --- |
| `w6-s3-cloudfront-architecture.svg` | Created for report | Explains the storage and static-delivery architecture |
| `w6-e01-s3-bucket-created.png` | `Pasted image 20260529132031.png` | S3 bucket baseline in `ap-southeast-1` |
| `w6-e02-raw-zeek-object-uploaded.png` | `Pasted image 20260529133301.png` | Raw Zeek-style dataset uploaded under `raw/zeek/` |
| `w6-e03-private-object-access-denied.png` | `Pasted image 20260529134108.png` | Direct raw object URL returned AccessDenied |
| `w6-e04-static-website-success.png` | `Pasted image 20260529141957.png` | S3 website endpoint rendered the static website |
| `w6-e05-selective-bucket-policy.png` | `Pasted image 20260529141829.png` | Bucket policy grants public read only to website files |
| `w6-e06-cloudfront-origin-website-endpoint.png` | `Pasted image 20260529150619.png` | CloudFront origin uses S3 website endpoint |
| `w6-e07-cloudfront-distribution-detail.png` | `Pasted image 20260529160527.png` | CloudFront distribution detail and default root object |
| `w6-e08-cloudfront-url-success.png` | `Pasted image 20260529160208.png` | CloudFront URL served the static website |
| `w6-e09-version-history.png` | `Pasted image 20260529163901.png` | Multiple versions of the same object key |
| `w6-e10-lifecycle-rule-enabled.png` | `Pasted image 20260529195041.png` | Lifecycle rule enabled for cleanup testing |

## Redaction Rules

Hide:

- AWS account ID
- Account alias/email
- Account ID inside ARN
- `RequestId` and `HostId` in AccessDenied responses
- Secrets, passwords, access keys, or private key material if they appear

Keep visible:

- Bucket name
- Object keys and prefixes
- Region
- CloudFront distribution domain
- S3 website endpoint
- Versioning status and object version behavior
- Lifecycle rule name and status
- Policy resources for `index.html` and `error.html`

## Evidence Interpretation

Current evidence supports:

- S3 bucket was created in `ap-southeast-1`.
- Object Ownership used ACLs disabled based on lab notes.
- Raw Zeek-style dataset object was uploaded under `raw/zeek/`.
- Private raw object access returned AccessDenied.
- Static website hosting was enabled for `index.html` and `error.html`.
- Bucket policy was selective and did not use a broad `bucket/*` resource.
- CloudFront delivered the static website using the S3 website endpoint as origin.
- S3 Versioning showed multiple versions of the same object key.
- Lifecycle rule was enabled for cleanup of old versions and delete markers.

Current evidence does not prove:

- Private S3 REST-origin delivery through CloudFront.
- WAF, ACM custom domain, signed URL, CI/CD, or production CDN hardening.
- Post-policy retest that `raw/zeek/conn_dataset.csv` remained private, although the displayed policy scope excludes it.

## Suggested Missing Evidence

If the user can provide more evidence later, the strongest additions would be:

- A screenshot after the selective bucket policy showing direct access to `raw/zeek/conn_dataset.csv` is still denied.
- A CloudFront cache behavior details page.
- A CloudFront distribution status page showing deployment complete.
- A cleanup screenshot showing the S3 bucket and CloudFront distribution removed after the lab.
- A future private-origin CloudFront implementation if the architecture is upgraded.

## Publication Checklist

Before switching Week 6 to `draft: false`, confirm:

- Daily time spent values are real or approved as estimates.
- Public screenshots are reviewed in `static/images/worklog/week6/`.
- No raw WSL/Windows path is embedded in the Hugo page.
- No account ID is visible in public screenshots.
- No request identifiers remain visible in the AccessDenied screenshot.
- No private keys, credentials, or access keys are visible.
- The page does not claim private S3 origin hardening.
- The CloudFront origin is described as the S3 static website endpoint.
- Week 7 plan points to local IDS lab architecture.
