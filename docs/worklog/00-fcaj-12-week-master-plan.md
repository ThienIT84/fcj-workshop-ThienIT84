# FCAJ 12-Week Worklog Master Plan

This document is the source of truth for the internship worklog.

The worklog must be FCAJ-first: each week should show an AWS service or cloud engineering capability, then connect it to the project use case, **Hybrid IDS/AI SOC System on AWS**. The IDS/AI project is the storyline, not a replacement for AWS service learning.

## Project Narrative

The internship project is a hybrid security monitoring system that combines:

- AWS cloud infrastructure and managed services.
- A local IDS lab for realistic network telemetry.
- Zeek and Suricata logs for dataset generation.
- AI-based detection and classification.
- A cost-aware AWS MVP migration path.

The report should not read like a disconnected list of AWS labs. It should show why each service was learned and how it supports a security monitoring workload.

Core storyline:

```text
Secure AWS account
-> build cloud network foundation
-> deploy compute and private workloads
-> add observability and secure operations
-> store and deliver data through S3/CloudFront
-> persist alert data in RDS
-> generate real IDS telemetry locally
-> build AI-ready datasets
-> design AI/backend APIs
-> migrate MVP components to AWS
-> review reliability, cost, and production improvements
```

## Status Labels

Use these labels consistently in weekly docs and final pages.

| Status | Meaning | Use when |
| --- | --- | --- |
| Implemented | Built and validated with evidence | AWS console, CLI output, logs, screenshots exist |
| Proof of Concept | Small demo or partial lab was built | Feature is not complete but behavior was tested |
| Researched | Studied and documented only | No implementation evidence yet |
| Proposed | Part of target architecture | Useful design, not deployed in MVP |
| Future Enhancement | Not done and not required for MVP | Improvement for production or next phase |

Do not claim production-grade deployment for services that were only researched or proposed.

## 12-Week Roadmap

| Week | Worklog title | AWS/service focus | Project tie-in | Source folders | Expected evidence |
| --- | --- | --- | --- | --- | --- |
| 1 | AWS Account, IAM & FCAJ Landing Zone | Root security, IAM users/groups, MFA, password policy, billing credits, region | Safe team access before building infrastructure | `FCAJ-Internship/00_Worklog/week_01`, `01_AWS-Labs/IAM` | Root MFA, no root access key, IAM users/groups, group policies, billing/region screenshots |
| 2 | VPC Network Foundation | VPC, subnets, route tables, IGW, public/private subnet, security groups | Cloud network segmentation for security workloads | `01_AWS-Labs/lab003_VPC` | VPC, subnet, route table, IGW, SG screenshots and network diagram |
| 3 | EC2, Bastion, NAT & Private Compute | EC2 public/private, key pair, NAT Gateway, EIP, bastion/private access | Compute pattern for backend/AI engine in private subnet | `lab003_VPC/Deploying Amazon EC2 Instance` | EC2 public/private, NAT route, SSH/private connection, route tables |
| 4 | Secure Operations & Observability | SSM Session Manager, EIC Endpoint, VPC Flow Logs, Reachability Analyzer, CloudWatch | Operate and troubleshoot infrastructure instead of only creating it | `Session Manager`, `Enable VPC Flow Logs`, `CloudWatch Monitoring & Alerting` | SSM target, endpoints, Flow Logs, Reachability Analyzer, CloudWatch metrics/alarm |
| 5 | S3 Storage & CloudFront Delivery | S3 bucket, object ownership, Block Public Access, versioning, lifecycle, static hosting, CloudFront | S3 for logs/datasets/artifacts; CloudFront for frontend/report/dashboard | `01_AWS-Labs/S3` | S3 bucket config, website endpoint, CloudFront URL, versioning/lifecycle |
| 6 | RDS PostgreSQL Data Layer | RDS PostgreSQL, DB subnet group, private subnet, RDS SG, EC2 client | Store alerts, risk scores, and dashboard data separately from raw logs | `01_AWS-Labs/Lab005_RDS`, `FCAJ_Project/Lab/RDS.md` | RDS VPC, DB subnet group, SG, PostgreSQL settings, EC2 client connection |
| 7 | Local IDS Lab Architecture | pfSense, Suricata, Zeek/ELK, Kali, Victim Ubuntu | Build realistic telemetry source instead of relying only on public datasets | `FCAJ_Project/Config Virtual Machine/Phase 0-7` | VM topology, IP plan, pfSense rules, Zeek/ELK running, connectivity tests |
| 8 | Telemetry Collection & Traffic Campaigns | AWS tie-in: prepare logs for S3/SQS ingestion | Generate normal and attack traffic, capture Zeek logs, create event diary | `Config Virtual Machine/Phase 8-9`, `Campain_crawl_dataset`, `List_attacks` | Campaign commands, attack commands, `conn.log`, `http.log`, `events.csv`, Suricata/ELK evidence |
| 9 | Dataset Factory & Data Quality Validation | AWS tie-in: data lake/artifact storage design on S3 | Convert raw logs into AI-ready CSV and validate quality | `Run_Crawl`, `Quy trình cào`, `profile_008`, `profile_009` | Parser command, output CSV, report JSON, row count, label/service distribution, `missed_bytes` check |
| 10 | AI Detection API & Backend Design | EC2/FastAPI runtime target, Docker concept | Design AI1/AI2A/AI2B, API contract, risk score, fusion layer | `Machine Learning`, `Planing/AI 2 & Integration.md`, `Machine Learning/Fusion layer.md` | API contract, feature table, model pipeline diagram, sample JSON response |
| 11 | AWS MVP Migration | EC2 Ubuntu backend, runtime setup, S3 artifacts, CloudFront/frontend | Move part of the SOC MVP to AWS with cost-aware deployment | `Migrate AWS/Phase 1`, `Phase 2`, `Phase 4` | EC2 runtime, SSH output, package versions, FastAPI health check, S3/CloudFront evidence |
| 12 | Reliability, Cost & Production Improvement | SQS/SNS, CloudWatch, Budgets, Parameter Store/Secrets Manager, cleanup | PoC/proposal for event-driven processing and production hardening | `ROADMAP_FCAJ_PROJECT`, `Cần cải thiện`, final docs | Budget, CloudWatch alarm, SQS/SNS if implemented, cleanup checklist, as-built vs target diagram |

## Service Coverage Matrix

| Service or capability | Primary week | Evidence expectation |
| --- | --- | --- |
| IAM, root security, MFA | Week 1 | Implemented evidence |
| VPC, subnet, route table, IGW, SG | Week 2 | Implemented evidence |
| EC2 public/private, NAT, EIP | Week 3 | Implemented evidence |
| SSM, EIC Endpoint, Flow Logs, CloudWatch | Week 4 | Implemented or troubleshooting evidence |
| S3, versioning, lifecycle, CloudFront | Week 5 | Implemented evidence |
| RDS PostgreSQL, DB subnet group | Week 6 | Implemented or lab evidence |
| Local IDS lab | Week 7 | Implemented evidence |
| Dataset generation and labeling | Week 8-9 | Implemented evidence |
| AI/API design | Week 10 | Design, PoC, or implementation evidence |
| EC2 MVP migration | Week 11 | Implemented evidence |
| SQS/SNS/Budgets/Secrets | Week 12 | PoC, implemented evidence, or target architecture |

## Project Coverage Matrix

| Project area | Week | Notes |
| --- | --- | --- |
| Security monitoring problem framing | Week 1 | Mention briefly, but keep AWS account/IAM as the service focus |
| AWS network foundation | Week 2-3 | Build VPC and private compute foundation |
| Cloud operations | Week 4 | Show troubleshooting and monitoring mindset |
| Data storage and delivery | Week 5-6 | Separate raw logs/artifacts from structured alert data |
| Local IDS telemetry | Week 7-8 | Show personal project depth |
| Dataset factory | Week 9 | Strong differentiator; include data quality checks |
| AI detection/backend | Week 10 | Keep as architecture and API layer, not the whole report |
| AWS MVP migration | Week 11 | Show real cloud deployment path |
| Production improvement | Week 12 | Show architect thinking without overstating implementation |

## As-Built MVP

Only include items that have evidence or strong notes:

- AWS IAM landing zone with root protection, admin user, IAM groups, password/MFA baseline.
- VPC lab with public/private subnets, route tables, Internet Gateway, Security Groups.
- EC2 public/private pattern, NAT Gateway, bastion/private access flow.
- Session Manager, EIC Endpoint, VPC Flow Logs, Reachability Analyzer, CloudWatch monitoring notes.
- S3 bucket, static website hosting, CloudFront distribution, versioning/lifecycle learning.
- RDS PostgreSQL private design with DB subnet group and EC2 client pattern.
- Local IDS lab with Kali, pfSense, Suricata, Victim Ubuntu, Zeek/ELK.
- Zeek-based dataset generation using `conn.log`, `http.log`, `events.csv`, and parser scripts.
- AI/backend design notes for AI1, AI2A, AI2B, FastAPI, and fusion layer.
- AWS MVP migration notes for EC2 backend runtime and S3/CloudFront frontend/artifacts.

## Target Production Architecture

Use this only as a proposed production improvement:

- ALB in front of API.
- Auto Scaling Group or ECS service for backend/AI workers.
- Private worker subnets.
- SQS + DLQ for asynchronous log processing.
- SNS or other notification service for alerts.
- Multi-AZ RDS for high availability.
- Secrets Manager or Parameter Store for credentials/configuration.
- WAF for edge/API protection.
- CloudWatch dashboard and alarms for API health, queue backlog, DLQ, CPU, and errors.
- Backup, retention, and cleanup policies.

## Evidence Rules

- Every week must include service evidence, project evidence, or both.
- Do not dump all screenshots at the bottom; place evidence near the claim it proves.
- Sensitive information must be hidden: passwords, MFA QR codes, access keys, full account IDs, private credentials.
- If a service was only researched, label it `Researched` or `Proposed`.
- If evidence is missing, write a short note explaining the intended validation.

## Final Worklog Page Pattern

Each weekly Hugo page should follow this structure:

```text
Objective
Service focus
Project context
Implementation summary
Validation and evidence
Issues and troubleshooting
Lessons learned
Next week plan
```

This keeps the report strong for FCAJ evaluators while preserving the Hybrid IDS/AI SOC storyline.

