---
title: "Worklog"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

This worklog documents **12 weeks** of internship at **Amazon Web Services Vietnam Co., Ltd.** (20/04/2026 – 12/07/2026), as part of the **AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform** project under the First Cloud Journey (FCAJ) program.

The program progressed through five main phases: building an AWS cloud foundation, setting up a data and delivery layer, constructing a local IDS lab and running traffic data collection campaigns, validating AI datasets and integrating the backend, and finally deploying an asynchronous cloud alert pipeline before archiving evidence and publishing the workshop report.

## Progress Overview

| Phase | Weeks | Main focus |
| --- | --- | --- |
| 🏗️ AWS Cloud Foundation | 1 – 4 | IAM, VPC, EC2/Bastion, CloudWatch/Session Manager |
| 🗄️ Data & Delivery Layer | 5 – 6 | RDS PostgreSQL, S3/CloudFront |
| 🔬 Local IDS Lab & Data Collection | 7 – 8 | pfSense/Kali/Zeek lab build + traffic campaigns & dataset generation |
| 🤖 AI Readiness & Backend | 9 – 10 | Model QA, FastAPI fusion layer |
| ☁️ Cloud Pipeline & Closure | 11 – 12 | SQS/Worker/RDS pipeline, cleanup, Hugo/Vercel deploy |

---

## Phase 1 — AWS Cloud Foundation

**[Week 1: AWS Account Security & IAM Access Baseline](1.1-week1/)**
Root user protection, MFA, billing alarm, Singapore region selection, administrator access baseline, team access planning, and least-privilege group/permission design. The lab phase practiced IAM users and group-based policies; actual project deployment later used AWS IAM Identity Center with permission sets and account assignments for team access control. Builder access was planned carefully and not granted broad deployment permissions in this phase.

**[Week 2: VPC Network Foundation & Segmentation](1.2-week2/)**
Custom VPC `10.10.0.0/16` with 2 public and 2 private subnets across 2 Availability Zones. Internet Gateway routing for public subnets, route table associations, and Security Group boundary design. Compute and NAT Gateway were deferred to Week 3.

**[Week 3: EC2, Bastion Access & NAT Private Egress](1.3-week3/)**
EC2 instances deployed in public and private subnets using the Week 2 VPC. Bastion-style SSH access from laptop through the public instance to private instances. Elastic IP allocation, NAT Gateway for private subnet outbound traffic, and key pair management. Operations tooling deferred to Week 4.

**[Week 4: Secure Operations & Observability](1.4-week4/)**
AWS Systems Manager Session Manager and EC2 Instance Connect Endpoint for private access without open SSH ports. VPC Flow Logs delivered to Amazon CloudWatch Logs. Metric filter for rejected network traffic and CloudWatch alarm captured at `ALARM` state with evidence.

---

## Phase 2 — Data & Delivery Layer

**[Week 5: RDS PostgreSQL Data Layer](1.5-week5/)**
Private Amazon RDS PostgreSQL instance placed in a DB Subnet Group across 2 AZs. Security Group access controlled from the EC2 client on port 5432. Created the `alerts` table schema with `severity`, `message`, and `anomaly_score` fields. Validated access using `psql` through the EC2 client in the pattern: Laptop → EC2 → private RDS.

**[Week 6: S3 Static Website, CloudFront Delivery & Object Lifecycle](1.6-week6/)**
Private S3 object storage for Zeek-style raw datasets. S3 static content hosting for selected public files. Amazon CloudFront distribution was configured to deliver static frontend content from the S3 origin. S3 Versioning enabled for object rollback protection. Lifecycle rule configured for cleanup of old versions and delete markers.

---

## Phase 3 — Local IDS Lab & Data Collection

**[Week 7: Local IDS Lab Build & Zeek Telemetry Validation](1.7-week7/)**
Built and validated a local Intrusion Detection lab with:
- **pfSense** as the virtual firewall/router separating network segments
- **Kali Linux** as the attacker host in the DMZ/OPT1 segment (`10.10.10.0/24`)
- **Ubuntu Victim** as the target host in the LAN segment (`192.168.1.0/24`)
- **Zeek sensor** monitoring LAN traffic from the same segment

Validated firewall rule enforcement (allow and default-deny), confirmed Zeek packet visibility using `tcpdump`, and inspected `conn.log` and `http.log` fields with UID correlation between network and HTTP layers. Exported Zeek telemetry into a structured CSV dataset and validated the output with Pandas. Kibana visualization was deferred.

**[Week 8: Traffic Campaign Execution & AI Dataset Workspace](1.8-week8/)**
Ran controlled traffic campaigns across the local IDS lab to generate labeled datasets:
- **Normal traffic:** 12 baseline profiles (P01–P12) covering typical network behavior
- **Attack traffic:** 10+ attack profiles (A01–A06, A08–A10 artifact-backed; A07 aggregate-only; A12 HTTP semantic)

Organized the dataset workspace for three AI components:
- **AI1** — Isolation Forest anomaly detector using 30 features from Zeek `conn.log`
- **AI2A** — Flow classifier using a 207,881-row Zeek conn dataset (77 columns, 53 model features)
- **AI2B** — HTTP semantic classifier (SQLi/XSS) using normalized URI/query evidence from Zeek `http.log`

A11 was not found in the workspace scan and was excluded from claims. A07 was referenced only in aggregate dataset documentation.

---

## Phase 4 — AI Readiness & Backend

**[Week 9: AI Dataset QA, Model Readiness & Validation Boundaries](1.9-week9/)**
QA matrix review across AI1, AI2A, and AI2B. Class distribution checks, split and holdout policy review, AI2A release-candidate evaluation, and AI2B freeze and blind-holdout boundary documentation. Produced a readiness summary with clear claim boundaries separating what was validated from what remains future work.

**[Week 10: AI Model Handoff, Backend API & Fusion Layer](1.10-week10/)**
FastAPI backend with `POST /api/events` as the main ingestion endpoint. AI routing logic directed flow evidence to AI1/AI2A and HTTP evidence to AI2B. Multi-model fusion layer produced `attack_type`, `risk_score`, `severity`, contributor list, and excluded-model flags. Model adapter contracts defined for each AI component.

---

## Phase 5 — Cloud Pipeline & Closure

**[Week 11: AWS Async Alert Pipeline: SQS, Worker, RDS & Idempotency](1.11-week11/)**
Deployed an asynchronous alert processing pipeline: `POST /api/events/http/async` → SQS main queue → `socai-ai-worker` consumer → RDS `final_alerts` table. AWS Secrets Manager used for RDS credentials. Validated D5-5 idempotency by replaying the same caller-provided `event_id` three times; `final_alerts` kept one row with the latest payload. `GET /api/alerts/latest` confirmed persisted alert retrieval through the API.

**[Week 12: Final Demo, Cleanup, Evidence Archive & Workshop Deployment](1.12-week12/)**
Final demo evidence confirmed a SQL Injection alert with critical severity, AI2B evidence, fusion confidence, and MITRE mapping was displayed through the deployed API and dashboard. Evidence archive and masking checklist completed. AWS cleanup executed in dependency order across compute, load balancing, queues, database, storage, secrets, CloudWatch, IAM, and network resources. RDS was deleted with a final snapshot, and Secrets Manager deletion was scheduled with a recovery window. CloudFront distribution, WAF Web ACL, and OAC were marked as deferred due to the CloudFront flat-rate pricing-plan constraint. Hugo workshop site published via Vercel as the final FCAJ submission.
