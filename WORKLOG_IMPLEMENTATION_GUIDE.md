# Worklog Implementation Guide

This document is a practical checklist for rewriting the FCAJ internship worklog.
Use it as a task board while replacing the sample Hugo content.

## Goal

Build a 12-week worklog that presents a clear learning and implementation journey:

```text
Cloud foundation
-> AWS network/compute/storage/database
-> observability and troubleshooting
-> local IDS lab
-> Zeek dataset pipeline
-> AI integration
-> AWS MVP migration
-> event-driven processing
-> hardening, cost, and final review
```

The worklog should not look like "one AWS service per week". Each week should show:

- Objective
- Completed work
- AWS services or technical tools used
- Evidence
- Issues and lessons learned
- Next step

## Important Writing Rule

Separate real implementation from design/proposal.

Use these labels consistently:

| Status | Meaning | Suggested wording |
| --- | --- | --- |
| Implemented | You actually built and tested it | Implemented and validated |
| Proof of Concept | You built a small demo or partial lab | Implemented as a proof of concept |
| Researched | You studied it but did not build fully | Researched and documented |
| Proposed | It belongs to the target architecture | Proposed for production architecture |
| Future Enhancement | Not done yet, but useful later | Planned as a future enhancement |

Avoid claiming a production-grade system if the MVP only uses simple resources.
For example:

```text
As-built MVP: one EC2 backend, S3/CloudFront frontend, local Zeek dataset pipeline.
Target architecture: ALB, ASG, private EC2 workers, SQS + DLQ, Multi-AZ RDS, WAF.
```

## Hugo File Mapping

Main worklog index:

```text
content/1-Worklog/_index.md
content/1-Worklog/_index.vi.md
```

Recommended weekly pages to create:

```text
content/1-Worklog/1.1-week1/_index.md
content/1-Worklog/1.2-week2/_index.md
content/1-Worklog/1.3-week3/_index.md
content/1-Worklog/1.4-week4/_index.md
content/1-Worklog/1.5-week5/_index.md
content/1-Worklog/1.6-week6/_index.md
content/1-Worklog/1.7-week7/_index.md
content/1-Worklog/1.8-week8/_index.md
content/1-Worklog/1.9-week9/_index.md
content/1-Worklog/1.10-week10/_index.md
content/1-Worklog/1.11-week11/_index.md
content/1-Worklog/1.12-week12/_index.md
```

Use Vietnamese versions too if you want bilingual content:

```text
_index.vi.md
```

## Weekly Template

Copy this structure for each week.

```markdown
---
title: "Week X - <Short Topic>"
date: 2026-XX-XX
weight: X
chapter: false
---

# Week X - <Short Topic>

## Objective

Briefly describe the purpose of the week.

## Completed Work

- Item 1
- Item 2
- Item 3

## Services and Tools

- AWS service or tool 1
- AWS service or tool 2
- Local tool if relevant

## Implementation Notes

Explain what was built, tested, configured, or documented.

## Evidence

- Screenshot:
- CLI output:
- Log/metric:
- Diagram:
- Repository/file:

## Issues and Troubleshooting

- Issue:
- Root cause:
- Fix:

## Lessons Learned

- Lesson 1
- Lesson 2

## Next Step

What this week prepares for in the following week.
```

## 12-Week Worklog Plan

# Week 1 – Cloud Foundation and Account Security

## Objective

Set up a secure AWS learning environment and establish the account, identity, Region, and cost-management foundations required for the project.

## Services and Concepts

* AWS Identity and Access Management (IAM)
* AWS Management Console
* AWS Regions
* Root account security
* Multi-Factor Authentication (MFA)
* IAM users, groups, roles, and policies
* Least-privilege access
* Billing and Free Tier monitoring
* AWS Budgets basics

## Completed Work

* Secured the AWS root account by enabling MFA and confirming that it would not be used for daily administrative work.
* Reviewed the differences between the AWS root user, IAM users, IAM groups, and IAM roles.
* Created IAM users and groups for administrative and project-related access.
* Assigned permissions through IAM groups instead of attaching unnecessary permissions directly to individual users.
* Created or reviewed an IAM role and practiced switching roles through the AWS Management Console.
* Studied the difference between identity-based policies and role trust policies.
* Reviewed why long-term access keys should be avoided when temporary role credentials or managed access methods are available.
* Performed a basic least-privilege test by allowing an IAM identity to perform an approved action and confirming that an unauthorized action returned an `AccessDenied` response.
* Selected `ap-southeast-1` as the primary AWS Region for the project because of its proximity to Vietnam, expected network latency, and availability of the required AWS services.
* Reviewed the AWS Billing dashboard and Free Tier usage information.
* Created or planned a basic cost alert to reduce the risk of unexpected AWS charges.
* Established a folder structure for worklog notes, architecture diagrams, screenshots, command outputs, and weekly evidence.

## Validation Performed

* Confirmed that MFA was enabled for the required account.
* Successfully signed in using a non-root identity.
* Successfully switched to the configured IAM role.
* Verified that an allowed IAM operation completed successfully.
* Verified that an unauthorized operation was denied according to the least-privilege policy.
* Confirmed that the AWS Console was operating in the selected primary Region.

## Evidence Collected

* Root account MFA status screenshot without exposing sensitive account information.
* IAM user and group configuration screenshots.
* IAM role permission policy screenshot.
* IAM role trust policy screenshot.
* Successful switch-role screenshot.
* Screenshot or CLI output showing an allowed operation.
* Screenshot showing an expected `AccessDenied` result.
* AWS Region selection screenshot.
* Billing, Free Tier, or Budget configuration screenshot.
* Short explanation of the least-privilege principle.
* Short explanation of why the root user and long-term access keys should not be used for daily work.

## Challenges and Lessons Learned

The main challenge was understanding the differences between an IAM user and an IAM role. An IAM user normally represents a long-term identity, while a role is assumed temporarily and provides temporary credentials.

The least-privilege test showed that secure access is not achieved by simply creating an IAM identity. Permissions must be limited to the exact actions and resources required by the user or application.

Region selection also affects latency, service availability, architecture consistency, and cost. Therefore, the primary Region should be selected before deploying the main infrastructure.

## Weekly Outcome

A secure AWS account baseline was established before deploying networking, compute, storage, or database resources. The account was configured to use non-root identities, MFA, role-based access, restricted permissions, a consistent primary Region, and basic cost monitoring.

## Worklog Summary

This week focused on establishing a secure AWS foundation before creating project infrastructure. The key lesson was that identity, permission, account protection, Region selection, and cost visibility must be considered before deploying compute or storage resources.


### Week 2 - Network Foundation Lab

Objective:
Build the AWS network foundation and connect it to the IDS/SOC project mindset.

Services and tools:

- VPC
- Public subnet
- Private subnet
- Route table
- Internet Gateway
- NAT Gateway concept
- Security Group

Completed work:

- Created a VPC lab with public and private subnets.
- Configured route tables and Internet Gateway.
- Studied NAT Gateway for private subnet outbound access.
- Compared AWS VPC design with the local IDS lab topology.

Evidence to collect:

- VPC diagram.
- Subnet list screenshot.
- Route table screenshot.
- Security Group screenshot.
- Short explanation of public vs private subnet.

Suggested worklog angle:

```text
This week connected AWS networking concepts with my local IDS lab, where traffic must flow through controlled network segments before being monitored.
```

### Week 3 - Compute and Secure Access

Objective:
Deploy compute resources and practice secure access patterns.

Services and tools:

- EC2
- Security Group
- SSH key pair
- EC2 Instance Connect Endpoint
- Session Manager
- CloudWatch basic metrics

Completed work:

- Launched EC2 instances for testing.
- Configured SSH access with a restricted source IP.
- Studied EC2 Instance Connect Endpoint and Session Manager.
- Reviewed CPU, memory, disk, and basic CloudWatch metrics.

Evidence to collect:

- EC2 instance screenshot.
- Security Group inbound rule screenshot.
- Successful SSH or Session Manager screenshot.
- CloudWatch EC2 metric screenshot.

Suggested worklog angle:

```text
This week focused on compute deployment and secure administration, which later supports the FastAPI/AI backend deployment.
```

### Week 4 - Storage and Static Delivery

Objective:
Use object storage for logs/artifacts and understand secure static delivery.

Services and tools:

- S3
- Bucket Policy
- S3 Block Public Access
- Versioning
- Lifecycle rules
- CloudFront
- Origin Access Control

Completed work:

- Created S3 buckets for static files, dataset artifacts, or log storage.
- Tested bucket policies and public access controls.
- Enabled versioning and reviewed lifecycle rules.
- Studied or configured CloudFront with private S3 origin using OAC.

Evidence to collect:

- S3 bucket screenshot.
- Block Public Access screenshot.
- Versioning/lifecycle screenshot.
- CloudFront distribution screenshot.
- Test showing direct S3 access denied and CloudFront access allowed, if implemented.

Suggested worklog angle:

```text
This week showed how S3 can serve two roles in the project: durable storage for logs/model artifacts and secure static hosting for a frontend.
```

### Week 5 - Database and Backend Persistence

Objective:
Design persistent storage for alerts, risk scores, and dashboard data.

Services and tools:

- RDS PostgreSQL
- DB subnet group
- Security Group
- Backup concept
- Private database endpoint

Completed work:

- Studied RDS PostgreSQL and how it differs from self-managed PostgreSQL on EC2.
- Designed a private RDS placement inside private subnets.
- Planned backend-to-database access through Security Groups.
- Identified what belongs in RDS: alerts, risk scores, classification results, users or dashboard metadata.

Evidence to collect:

- RDS configuration screenshot if labbed.
- DB subnet group screenshot if created.
- Security Group rule screenshot.
- Connection test from backend/EC2 if implemented.
- Note explaining why raw logs go to S3 but final alert data goes to RDS.

Suggested worklog angle:

```text
This week focused on separating raw log storage from structured alert storage. S3 is better for large files, while RDS is better for queryable alert records.
```

### Week 6 - Observability and Troubleshooting

Objective:
Learn how to debug network and system issues using AWS observability tools.

Services and tools:

- VPC Flow Logs
- Reachability Analyzer
- CloudWatch Logs
- CloudWatch Alarm

Completed work:

- Enabled or studied VPC Flow Logs for network visibility.
- Used Reachability Analyzer to reason about connectivity.
- Created or planned CloudWatch logs/alarms for EC2 or backend monitoring.
- Documented troubleshooting cases such as SSH failure, Security Group blocking, or route table misconfiguration.

Evidence to collect:

- Flow Logs screenshot.
- Reachability Analyzer result.
- CloudWatch Alarm screenshot.
- Troubleshooting note with issue/root cause/fix.

Suggested worklog angle:

```text
This week shifted from building resources to operating them. The focus was finding evidence when traffic fails or a service becomes unhealthy.
```

### Week 7 - Local IDS Lab Design

Objective:
Build the local IDS lab topology used to generate realistic network telemetry.

Services and tools:

- Kali
- pfSense
- Suricata
- Zeek
- ELK/Kibana
- Ubuntu Victim
- VMware network segments

Completed work:

- Defined the lab topology.
- Assigned roles and IPs:
  - Kali: `10.10.10.10`
  - pfSense LAN: `192.168.1.1`
  - Victim Ubuntu: `192.168.1.10`
  - ELK + Zeek: `192.168.1.20`
- Configured or documented traffic flow from Kali through pfSense to Victim.
- Prepared Zeek/ELK for log capture.

Evidence to collect:

- Topology diagram.
- VM network screenshots.
- Ping/curl connectivity tests.
- Zeek or ELK service screenshot.

Suggested worklog angle:

```text
This week established the local source of truth for the dataset. Instead of relying only on public datasets, the project generates telemetry from a controlled lab.
```

### Week 8 - Dataset Collection Pipeline

Objective:
Generate clean IDS datasets from Zeek logs and event labels.

Services and tools:

- Zeek `conn.log`
- Zeek `http.log`
- `events.csv`
- Python parser
- Locust
- Paramiko/sshpass
- nmap
- hydra
- curl/wget

Completed work:

- Generated normal traffic such as HTTP, SSH, DNS, file download, SCP, and HTTPS metadata.
- Generated controlled attack traffic such as port scan, brute force, and web attack payloads.
- Captured logs with Zeek.
- Created event diary files for labeling.
- Parsed raw logs into AI-ready datasets.
- Applied cleanup rules such as dropping IPv6, filtering IPs, and removing flows with `missed_bytes`.

Evidence to collect:

- Campaign command screenshot or log.
- Zeek `conn.log`/`http.log` sample.
- `events.csv` sample.
- Parser output CSV.
- Dataset summary: row count, label count, service distribution.
- Report showing `missed_bytes = 0` or cleanup behavior.

Suggested worklog angle:

```text
This week converted raw lab traffic into a dataset pipeline. The main result was not only data volume, but a repeatable process with labels, parser output, and quality checks.
```

### Week 9 - AI Pipeline and API Design

Objective:
Design the AI detection layer and API contract for the SOC MVP.

Services and tools:

- Isolation Forest
- XGBoost or Random Forest
- FastAPI
- Docker
- OpenAPI/Swagger
- Fusion Layer concept

Completed work:

- Designed AI1 for anomaly detection.
- Designed AI2A for network-level attack classification from `conn.log`.
- Designed AI2B for HTTP/web attack detection from `http.log`.
- Defined expected API input/output.
- Planned risk scoring and fusion layer behavior.
- Studied explainability with SHAP/LIME as a possible improvement.

Evidence to collect:

- API contract screenshot or JSON example.
- Model pipeline diagram.
- FastAPI `/docs` screenshot if implemented.
- Sample prediction response.
- Notes explaining false positive reduction and risk scoring.

Suggested worklog angle:

```text
This week connected the dataset pipeline with the detection layer. The focus was turning network telemetry into classification, risk score, and explainable output for a dashboard.
```

### Week 10 - AWS MVP Migration

Objective:
Prepare or deploy the backend MVP on AWS.

Services and tools:

- EC2 Ubuntu
- Security Group
- Elastic IP concept
- Miniconda
- Python 3.10
- FastAPI
- S3 artifact storage
- CloudWatch logs if implemented

Completed work:

- Created an EC2 backend environment for the SOC MVP.
- Installed runtime tools such as Git, rsync, curl, tmux, build tools, Miniconda, and Python.
- Prepared project directory such as `~/soc-mvp`.
- Copied or planned backend/model artifacts.
- Tested outbound Internet and API health check if available.

Evidence to collect:

- EC2 instance screenshot.
- SSH terminal showing OS, CPU, RAM, disk.
- Installed runtime/version screenshots.
- FastAPI health check result if implemented.
- Security Group rules.
- S3 artifact screenshot if used.

Stronger output wording if implemented:

```text
Deployed FastAPI backend on EC2, configured runtime environment, validated `/health` endpoint, and documented the deployment steps for repeatability.
```

Suggested worklog angle:

```text
This week moved the project from local development toward an AWS-hosted MVP. The implementation stayed cost-conscious while preserving a path to production architecture.
```

### Week 11 - Event-Driven Processing and Alerting

Objective:
Decouple log ingestion from AI processing and create alert notification.

Services and tools:

- SQS
- SQS DLQ
- SNS
- CloudWatch Alarm
- S3 Event Notification, optional
- FastAPI ingestion API, optional

Choose one MVP flow to implement:

```text
FastAPI ingestion
-> SQS
-> AI worker
-> RDS or S3 result
-> SNS alert
-> CloudWatch log/alarm
```

Alternative batch flow:

```text
S3 raw log upload
-> S3 Event Notification
-> SQS
-> batch worker
-> processed result
```

Completed work:

- Created an SQS queue for log/detection jobs.
- Created an SNS topic for high-risk alerts.
- Connected worker behavior conceptually or through a small proof of concept.
- Added a DLQ or documented retry behavior.
- Tested success and failure cases if possible.

Evidence to collect:

- SQS queue screenshot.
- Message send/receive screenshot.
- SNS email subscription screenshot.
- Received alert email screenshot.
- DLQ screenshot if tested.
- CloudWatch metric/alarm for queue depth or DLQ messages.

Suggested worklog angle:

```text
This week improved the architecture from direct processing to event-driven processing. SQS helps absorb traffic bursts and prevents the AI worker from blocking ingestion.
```

### Week 12 - Hardening, Cost, and Final Review

Objective:
Review the MVP, document trade-offs, and prepare the final report.

Services and tools:

- AWS Budgets
- SSM Parameter Store or Secrets Manager
- CloudWatch Dashboard
- Cleanup checklist
- Cost estimation
- Architecture diagram

Completed work:

- Created or documented cost-control practices with AWS Budgets.
- Moved configuration or secrets to Parameter Store/Secrets Manager as a proof of concept.
- Created final architecture diagrams.
- Separated as-built architecture from target production architecture.
- Wrote cleanup checklist.
- Prepared screenshots and evidence for the report.

Evidence to collect:

- AWS Budget screenshot.
- Parameter Store/Secrets Manager screenshot if implemented.
- Final architecture diagram.
- Cost estimate.
- Cleanup checklist.
- Final demo checklist.

Suggested worklog angle:

```text
The final week focused on operational maturity: cost awareness, secret handling, cleanup, and documenting how the MVP could evolve into a production-grade architecture.
```

## Evidence Checklist

Use this list while collecting proof for the report.

### AWS Evidence

- IAM user/group/role/switch role screenshots
- VPC, subnet, route table, IGW screenshots
- EC2 instance and Security Group screenshots
- Session Manager or SSH connection proof
- S3 bucket, policy, versioning, lifecycle screenshots
- CloudFront distribution and OAC screenshots
- RDS setup or design screenshot
- VPC Flow Logs screenshot
- Reachability Analyzer result
- CloudWatch Logs/Alarm screenshots
- SQS queue, message, DLQ screenshots
- SNS subscription and alert email screenshot
- AWS Budgets screenshot
- Parameter Store/Secrets Manager screenshot

### Local IDS Evidence

- VMware topology screenshot
- pfSense interface/rule screenshot
- Suricata alert screenshot if available
- Zeek running/capture screenshot
- ELK/Kibana screenshot if available
- Kali normal/attack command output
- Victim web/SSH service output

### Dataset Evidence

- `conn.log` sample
- `http.log` sample
- `events.csv` sample
- Parser command
- Output CSV
- Dataset summary
- `missed_bytes` check
- Label distribution
- Service/port distribution

### AI/API Evidence

- API contract
- FastAPI `/docs`
- `/health` response
- Sample prediction response
- Model pipeline diagram
- Risk scoring example
- Fusion layer explanation

## Architecture Decision Table

Add this table to the Proposal or Workshop section.

| Decision | Choice | Reason | Trade-off |
| --- | --- | --- | --- |
| Backend runtime | EC2 | Suitable for Python ML/FastAPI and keeping models in memory | Need to manage OS, patching, and scaling |
| Raw log storage | S3 | Cheap, durable, suitable for large log files and model artifacts | Not ideal for transactional queries |
| Final alert storage | RDS PostgreSQL | Structured queries for dashboard and alert filtering | Higher cost than object storage |
| Async processing | SQS Standard | Decouples ingestion from AI worker and handles bursts | Worker must handle duplicate delivery |
| Failed messages | SQS DLQ | Makes failure visible and recoverable | Adds operational complexity |
| Alert notification | SNS | Simple email notification for high-risk alerts | Not a full incident management platform |
| Frontend delivery | CloudFront + S3 OAC | Secure static delivery with private S3 origin | Requires CloudFront invalidation/cache management |
| Secret/config storage | Parameter Store | Simple and low-cost configuration storage | Secrets Manager has better rotation features |
| Monitoring | CloudWatch | Native AWS logs, metrics, and alarms | Custom dashboards may require extra setup |

## As-Built vs Target Architecture

Use this section to stay honest and still show strong architecture thinking.

### As-Built MVP

Write only what you actually implemented or validated.

Example:

```text
- Local IDS lab with Kali, pfSense, Suricata, Victim Ubuntu, and Zeek/ELK.
- Zeek-based dataset generation pipeline using conn.log, http.log, events.csv, and parser scripts.
- EC2 Ubuntu backend environment prepared for FastAPI/AI runtime.
- S3 bucket for static/frontend or artifact storage.
- CloudFront with private S3 origin using OAC.
- Basic CloudWatch monitoring and troubleshooting evidence.
```

### Target Production Architecture

Write what should be added for production.

Example:

```text
- ALB in front of backend API.
- Auto Scaling Group across multiple Availability Zones.
- Private EC2 workers or ECS service for AI processing.
- SQS + DLQ for resilient asynchronous processing.
- RDS Multi-AZ for higher database availability.
- Secrets Manager for credential rotation.
- WAF for edge/API protection.
- CloudWatch dashboard and alarms for queue backlog, DLQ, CPU, and API errors.
- Backup, retention, and cleanup policies.
```

## Minimum Excellent-Level Deliverables

Aim to finish these before writing the final version:

- One clear architecture diagram with numbered data flow.
- One end-to-end demo path.
- One service decision/trade-off table.
- One security checklist.
- One failure test, such as SQS DLQ or Security Group blocked connection.
- One monitoring test, such as CloudWatch Alarm entering `ALARM`.
- One cost estimate or AWS Budget screenshot.
- One cleanup checklist.
- Screenshots, CLI outputs, logs, and metrics for each major claim.

## Suggested End-to-End MVP Demo

If time is limited, implement this small but convincing flow:

```text
1. Generate or select one sample Zeek event/dataset row.
2. Send it to a FastAPI endpoint or directly to SQS.
3. Worker consumes the message.
4. Worker produces a mock or real risk score.
5. Result is written to S3 or RDS.
6. High-risk result triggers SNS email.
7. CloudWatch shows logs/metrics for the run.
```

This is enough to prove the architecture pattern without pretending the entire production SOC is complete.

## Final Review Checklist

Before publishing:

- [ ] Each week has objective, completed work, evidence, issues, lessons, and next step.
- [ ] Basic services are grouped into meaningful project phases.
- [ ] No week is only "I learned service X".
- [ ] Claims are marked as implemented, proof of concept, researched, proposed, or future enhancement.
- [ ] Screenshots are copied into `static/images/...` and referenced from the Hugo pages.
- [ ] `public/` is not manually edited.
- [ ] `hugo server` renders all worklog pages correctly.
- [ ] Architecture diagrams are readable on desktop and mobile.
- [ ] Cleanup and cost notes are included.

