---
title: "Introduction"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---



## Problem Statement

In traditional security monitoring systems, logs are often generated from various sources such as firewalls, IDS/IPS, network sensors, or servers. Without a centralized pipeline, log collection and processing face many challenges:

- Logs are scattered across multiple machines or different tools.
- Sending logs from Local Lab to Cloud is not automated.
- Backend can be overloaded if processing all events directly.
- Raw evidence is difficult to store fully for audit or re-analysis.
- Dashboard lacks centralized data to display alerts.
- System is hard to scale without separating event reception, processing, and storage layers.

This workshop solves the problem by building an architecture capable of:

**Log/evidence collection:** Receiving data from Local Lab via Zeek Collector or Tailer.

**Event reception:** EC2 Backend receiving requests from collector or dashboard.

**Asynchronous processing:** Amazon SQS separating ingest and processing streams to prevent backend overload.

**Raw evidence storage:** Amazon S3 Data Bucket storing original data for audit and reprocessing when needed.

**Processing results storage:** Amazon RDS PostgreSQL storing alerts, event metadata, and analysis results.

**Alert display:** React Dashboard displaying alerts for admins or SOC analysts via API/WebSocket.

## Solution Architecture

The system is designed with a hybrid architecture between Local Security Lab and AWS Cloud. Local Lab generates logs and evidence, then the collector sends data to the backend running on EC2. The backend receives events, writes raw evidence to S3, and sends messages to SQS for workers to process asynchronously.

After processing, data is stored in RDS PostgreSQL and displayed on the dashboard via API or WebSocket.

Overall system flow:

```text
Local Lab
-> Zeek Collector / Tailer
-> EC2 Backend
-> SQS + S3 Data Bucket
-> Worker / Fusion
-> RDS PostgreSQL
-> API / WebSocket
-> React Dashboard
```

### Local Lab

Local Lab is where input data for the SOC system is generated. In this model, Local Lab includes attacker, firewall/router, victim machine, Zeek sensor, and collector/tailer.

Main components:

- Kali Linux simulating attacker or generating test traffic.
- pfSense acting as firewall/router.
- Victim machine generating traffic for the sensor to observe.
- Zeek sensor analyzing network traffic and generating logs.
- Zeek Collector/Tailer reading logs and sending evidence to cloud backend.

### Frontend Interface

Users access the system through a dashboard on a web browser.

Dashboard access flow:

```text
Admin / Analyst
-> HTTPS Dashboard
-> CloudFront
-> S3 Static Frontend
```

Frontend is deployed as a static web application. Amazon S3 stores static assets, CloudFront distributes content via CDN, and WAF can be used to filter malicious requests at the edge layer if configured.

Dashboard helps admins or analysts monitor:

- Alert list.
- Severity level.
- Log source.
- Detection time.
- Event details.
- Processing status.

### Request Handling

Requests from dashboard or collector will be sent to the backend API.

In the target architecture, requests may go through Application Load Balancer and Target Group before reaching EC2 backend. Backend runs FastAPI to handle REST API, WebSocket, or event ingest endpoints.

Main components:

- Application Load Balancer routing traffic to backend.
- Target Group managing EC2 backend instances.
- EC2 Backend running FastAPI API service.
- Auto Scaling Group can scale the number of backend instances when needed.

At the current stage, if ALB or Auto Scaling Group is not fully configured, backend can be tested directly via EC2 to complete the core flow first.

### Backend Processing

Backend Processing Layer is responsible for processing events after the backend receives data from Local Lab.

Main processing flow:

```text
EC2 Backend
-> S3 Data Bucket
-> SQS Queue
-> Worker / Fusion
-> RDS PostgreSQL
```

EC2 Backend will receive events, validate payload, generate `event_id`, write raw evidence to S3 Data Bucket, and send messages to SQS. Worker or Fusion component reads messages from SQS, processes events, and saves results to RDS PostgreSQL.

This design helps the system:

- Reduce load on backend API.
- Separate event reception and event processing processes.
- Limit data loss when workers fail.
- Can scale workers independently.
- Save raw evidence for audit purposes.

### Data Layer

Data Layer includes services for storing raw data and processed data.

Amazon S3 is used to store raw evidence from Local Lab. An example object key could be:

```text
raw/zeek/http/2026/07/06/evt-xxxx.json
```

Amazon RDS PostgreSQL is used to store:

- Event metadata.
- Alerts.
- Detection results.
- Severity.
- Timestamp.
- Source log type.
- Processing status.

In the demo stage, RDS PostgreSQL can be deployed in Single-AZ mode for cost optimization. Multi-AZ is a production-style extension and should not be claimed as deployed if not actually enabled.

### Network and Security

The system is deployed in one AWS Region and a dedicated VPC. Inside the VPC, subnets are divided by role to separate public traffic, backend runtime, and database.

Main network components:

- VPC as the private network for the entire system.
- Public subnet containing NAT Gateway or ALB if available.
- Private subnet containing backend EC2 or workers.
- Private DB subnet containing RDS PostgreSQL.
- Internet Gateway for public subnet to connect to the Internet.
- NAT Gateway for private subnet to go outbound to the Internet if available.
- Security Group controlling inbound and outbound traffic.
- VPC Endpoint allowing access to S3 without going through the Internet.

System security baseline includes:

- IAM Role for EC2 Backend.
- Secrets Manager for storing sensitive information.
- KMS for encrypting secrets or data if configured.
- Security Group restricting SSH, backend port, and database port.
- No hard-coded AWS access keys in source code or `.env`.

### Monitoring and Notification

Monitoring helps the team check backend status, pipeline, and runtime errors.

Main components:

- CloudWatch used to monitor logs and metrics.
- CloudTrail recording API activity in the AWS account.
- SNS can be used to send alerts via Email/SMS.

If the backend fails, the team can check logs directly on EC2 with:

```bash
sudo journalctl -u socai-backend -n 80 --no-pager
```

If CloudWatch Logs is configured, backend logs can be pushed to CloudWatch for centralized monitoring.

## Architect

![Hybrid SOC Architecture](/images/5-Workshop/5.1-Overview-and-Architecture/w-trang-01-architecture-overview.jpg)

## Key Technologies

In this workshop, you will work with the following main AWS services and technologies:

**Amazon EC2:** Runs FastAPI Backend, API service, WebSocket, or worker logic.

**Amazon VPC:** Provides private network for the system, including subnets, route tables, Internet Gateway, NAT Gateway, and Security Group.

**Amazon S3:** Stores static frontend and raw evidence from Local Lab.

**Amazon SQS:** Message queue used for asynchronous event processing between backend and workers.

**Amazon RDS PostgreSQL:** Stores event metadata, alerts, and processing results.

**AWS IAM:** Manages access permissions for EC2 Backend via IAM Role.

**AWS Secrets Manager:** Stores sensitive information like database credentials.

**AWS KMS:** Supports encrypting secrets or data if required by the system.

**Amazon CloudFront:** Distributes frontend dashboard via CDN.

**AWS WAF:** Protects frontend or API from malicious requests if configured.

**Application Load Balancer:** Routes traffic to EC2 backend via Target Group.

**Auto Scaling Group:** Scales the number of EC2 Backend instances in a production-style architecture.

**Amazon CloudWatch:** Monitors logs, metrics, and backend activity status.

**AWS CloudTrail:** Records API activities in the AWS account.

**Amazon SNS:** Sends alerts via Email/SMS if the system has notifications enabled.

## Current Deployment Status

Some components in the diagram are target architecture or production-style extensions. At the current stage, the workshop prioritizes completing the core flow first:

```text
Local Lab
-> EC2 Backend
-> SQS / S3
-> Worker / RDS
-> Dashboard
```

Components like ALB, Target Group, Auto Scaling Group, CloudTrail, and SNS should only be noted as extensions if there is no actual configuration evidence on AWS Console.

The backend is currently being updated with source code to add boto3 integration for S3 and SQS. Therefore, the next section will focus on checking EC2 runtime, systemd service, `/health` endpoint, and placing a  for the `/api/events/http/async` endpoint until the new source is ready.
