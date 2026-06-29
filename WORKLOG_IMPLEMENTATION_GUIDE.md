# Worklog Implementation Guide

This guide explains how to implement the FCAJ internship worklog.

Primary planning source:

```text
docs/worklog/00-fcaj-12-week-master-plan.md
```

The worklog should be **FCAJ-first**. Each week must show AWS service learning or cloud engineering capability, then connect it to the project use case: **Hybrid IDS/AI SOC System on AWS**.

## Goal

Build a 12-week report that shows both:

- AWS service learning and implementation.
- A coherent security monitoring project that gives the service work meaning.

The report should not become only AI/IDS content. The IDS/AI system is the use case, while AWS services remain the weekly backbone.

## Hugo File Mapping

Main worklog index:

```text
content/1-Worklog/_index.md
content/1-Worklog/_index.vi.md
```

Weekly pages already follow this repo's casing:

```text
content/1-Worklog/1.1-Week1/_index.md
content/1-Worklog/1.2-Week2/_index.md
content/1-Worklog/1.3-Week3/_index.md
content/1-Worklog/1.4-Week4/_index.md
content/1-Worklog/1.5-Week5/_index.md
content/1-Worklog/1.6-Week6/_index.md
content/1-Worklog/1.7-Week7/_index.md
content/1-Worklog/1.8-Week8/_index.md
content/1-Worklog/1.9-Week9/_index.md
content/1-Worklog/1.10-Week10/_index.md
content/1-Worklog/1.11-Week11/_index.md
content/1-Worklog/1.12-Week12/_index.md
```

Do not manually edit `public/`.

## Status Labels

| Status | Meaning | Suggested wording |
| --- | --- | --- |
| Implemented | Built and validated with evidence | Implemented and validated |
| Proof of Concept | Small demo or partial lab | Implemented as a proof of concept |
| Researched | Studied and documented only | Researched and documented |
| Proposed | Target architecture only | Proposed for production architecture |
| Future Enhancement | Later improvement | Planned as a future enhancement |

Use these labels to avoid overstating unfinished services.

## 12-Week FCAJ-First Plan

| Week | Title | Service focus | Project tie-in |
| --- | --- | --- | --- |
| 1 | AWS Account Security & IAM Access Baseline | Root security, IAM, MFA onboarding, groups, billing credits, region | Secure team access before building infrastructure |
| 2 | VPC Network Foundation | VPC, subnet, route table, IGW, SG | Network segmentation for security workloads |
| 3 | EC2, Bastion, NAT & Private Compute | EC2 public/private, NAT Gateway, EIP, key pair | Private compute pattern for backend/AI engine |
| 4 | Secure Operations & Observability | SSM, EIC Endpoint, VPC Flow Logs, Reachability Analyzer, CloudWatch | Operate and troubleshoot cloud infrastructure |
| 5 | S3 Storage & CloudFront Delivery | S3, versioning, lifecycle, static hosting, CloudFront | Store logs/datasets/artifacts and serve frontend/report |
| 6 | RDS PostgreSQL Data Layer | RDS, DB subnet group, private subnet, RDS SG | Store structured alert/risk/dashboard data |
| 7 | Local IDS Lab Architecture | pfSense, Suricata, Zeek/ELK, Kali, Victim | Generate realistic security telemetry |
| 8 | Telemetry Collection & Traffic Campaigns | AWS tie-in: prepare logs for S3/SQS ingestion | Normal/attack campaigns, Zeek logs, event diary |
| 9 | Dataset Factory & Data Quality Validation | AWS tie-in: S3 data lake/artifact storage | Parser, CSV, labels, `missed_bytes`, quality checks |
| 10 | AI Detection API & Backend Design | EC2/FastAPI runtime target, Docker concept | AI1/AI2A/AI2B, API contract, fusion layer |
| 11 | AWS MVP Migration | EC2 Ubuntu backend, runtime, S3 artifacts, CloudFront | Move SOC MVP components to AWS |
| 12 | Reliability, Cost & Production Improvement | SQS/SNS, CloudWatch, Budgets, Secrets/Parameter Store, cleanup | PoC/proposal for production hardening |

## Weekly Page Template

Use this structure for final Hugo pages:

```markdown
---
title: "Week X - <Title>"
date: 2026-XX-XX
weight: X
chapter: false
pre: " <b> 1.X. </b> "
---

# Week X - <Title>

## Objective

## Service Focus

## Project Context

## Implementation Summary

## Validation and Evidence

## Issues and Troubleshooting

## Lessons Learned

## Weekly Outcome

## Next Week Plan
```

## Evidence Rules

- Put screenshots near the claim they prove.
- Use 5-8 strong evidence items per week instead of dumping every screenshot.
- Hide passwords, MFA QR codes, access keys, full account IDs, and private credentials.
- Store images under `static/images/worklog/weekX/`.
- Reference images from Hugo with `/images/worklog/weekX/<file>.png`.

Example:

```markdown
![Root MFA enabled](/images/worklog/week1/01-root-mfa-enabled.png)
```

## Excellent-Level Checklist

Each week should answer:

- What AWS service/capability was learned?
- Why does it matter for the Hybrid IDS/AI SOC project?
- What was implemented or validated?
- What evidence proves it?
- What issue or trade-off was discovered?
- How does this prepare the next week?

Strong wording:

```text
Implemented and validated
Configured and tested
Verified using AccessDenied
Troubleshot using Reachability Analyzer
Measured with CloudWatch
Documented trade-offs
```

Weak wording:

```text
Learned
Explored
Prepared
Tried
```

Use weak wording only for research or proposed work.

## As-Built vs Target Architecture

Use this distinction in Week 11-12 and Proposal:

As-built MVP:

```text
What was actually built or validated with evidence.
```

Target architecture:

```text
What should exist in production, such as ALB, ASG, SQS DLQ, Multi-AZ RDS, WAF, and Secrets Manager.
```

This is better than pretending the MVP is production-grade.

## Final Review Before Publishing

- [ ] 12 weeks have service coverage, not only AI content.
- [ ] Week 1 starts with a single-account IAM/access baseline.
- [ ] Week 2 starts VPC.
- [ ] Claims are labeled as implemented, PoC, researched, proposed, or future enhancement.
- [ ] Screenshots are stored in `static/images/worklog/weekX/`.
- [ ] Hugo paths use existing repo casing: `1.1-Week1`, not lowercase `week1`.
- [ ] `public/` is not manually edited.
- [ ] Hugo build/server renders the pages correctly.
