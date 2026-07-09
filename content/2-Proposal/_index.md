---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform
## A Hybrid Cloud SOC Platform for Multi-Model AI Security Monitoring and Response

### 1. Executive Summary
The AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform is proposed as a hybrid cloud security monitoring platform that connects a Local Security Lab with AWS infrastructure for log collection, event processing, AI inference, and real-time SOC dashboard visualization. The system will combine telemetry from pfSense, Zeek, Suricata, and a lab collector/tailer with AWS services such as CloudFront, WAF, ALB, EC2, SQS, RDS PostgreSQL, S3, CloudWatch, CloudTrail, SNS, IAM, Secrets Manager, and KMS.

The final goal is to build a production-style architecture that receives security events over HTTPS, normalizes events at the backend ingestion API, pushes messages into SQS for asynchronous processing, runs specialized AI models, and combines model outputs through a Fusion Layer. Final alerts will be stored in RDS/S3 and displayed on the dashboard with severity, risk score, confidence score, MITRE ATT&CK mapping, evidence, and decision flow.

The platform will use three main AI analysis tracks: AI1 for anomaly detection on network flows, AI2A for network attack classification from Zeek `conn.log`, and AI2B for HTTP semantic web attack detection such as SQL Injection or Cross-Site Scripting from Zeek `http.log`. The Fusion Layer will combine AI outputs and rule-based evidence into final alerts for investigation and incident response.

### 2. Problem Statement
*Current Problem*

In a small SOC or security research lab, security data is often fragmented across Zeek `conn.log`, Zeek `http.log`, Suricata alerts, pfSense logs, and manual pentest notes. Manual log review slows down incident detection, makes time-based evidence correlation difficult, and does not scale when attack campaigns generate larger event volumes.

Traditional rule-based systems can detect some clear indicators, but they can miss anomalous behavior or encoded/obfuscated web attacks. On the other hand, a single AI model usually lacks context, is harder to explain, and can create false positives. An effective SOC dashboard needs to combine multiple evidence sources, multiple AI models, and a fusion layer that can produce explainable final alerts.

*Solution*

The project proposes a hybrid cloud architecture where the Local Collector does not send messages directly to SQS. Instead, the collector will send events over HTTPS to CloudFront/WAF, through ALB, and into the backend ingestion API. The backend will authenticate, validate, normalize, and route events before pushing messages into SQS. A worker service in private subnets will consume messages from SQS, run AI inference, execute Fusion, and write final alerts to RDS PostgreSQL/S3.

The React/Vite dashboard will be built as static files and stored in a private S3 frontend bucket. CloudFront will use Origin Access Control to read objects from S3 and serve the dashboard over HTTPS. The backend API and WebSocket service will be exposed through ALB, while the worker tier will stay private and communicate only with SQS, RDS, S3, Secrets Manager, and CloudWatch through appropriate IAM permissions.

*Benefits and Return on Investment*

The solution will help the team build an end-to-end SOC platform from lab attack traffic to final alerts on a dashboard. It reduces manual log review, creates structured evidence, helps analysts understand why an alert was generated, and provides a foundation for improving AI models over time. Architecturally, separating the backend tier from the worker tier allows the team to control real-time API/WebSocket workloads separately from AI processing workloads, and to scale based on request rate, CPU, or queue backlog.

### 3. Solution Architecture
This proposal describes the final target architecture of the project. Implementation will be divided into phases to reduce risk, validate each layer, and control cost.

#### 3.1. Target Architecture Diagram

![Hybrid SOC Target Architecture](/images/2-Proposal/hybrid_soc_target_architecture.jpg)

*Figure 1. Target architecture of the hybrid cloud SOC platform. The diagram shows the flow from Local Lab, CloudFront/WAF, ALB, backend private compute, SQS worker, AI/Fusion, RDS/S3, to monitoring and governance.*

Target processing flow:

```text
Local Collector
  -> CloudFront / AWS WAF
  -> Application Load Balancer
  -> Backend ingestion API
  -> Amazon SQS
  -> Worker service
  -> AI1 / AI2A / AI2B + Fusion Layer
  -> Amazon RDS for PostgreSQL / Amazon S3
  -> Dashboard API + WebSocket
  -> SOC Dashboard
```

#### 3.2. Local Lab and Log Sources

The Local Security Lab will include an attacker machine, pfSense firewall, victim machine, Zeek sensor, and Zeek Collector/Tailer. Attack traffic will pass through pfSense and victim services, then be observed by Zeek to produce `conn.log` and `http.log`. Suricata or rule-based evidence can be added to provide additional context for the Fusion Layer.

The Collector/Tailer will read logs, normalize required fields, attach timestamp/correlation identifiers, and send events over HTTPS to the AWS ingestion endpoint. The collector does not need AWS credentials to write directly to SQS, reducing credential exposure risk in the lab environment.

#### 3.3. Edge, Frontend, and CloudFront/WAF

SOC users will access the dashboard through CloudFront over HTTPS. The React/Vite frontend will be built into static files and stored in a private S3 frontend bucket. The frontend bucket will not be public; CloudFront will use Origin Access Control to read objects from S3 and securely serve the dashboard.

AWS WAF will be placed at the public entry layer to apply Web ACLs and managed rules to incoming requests. CloudFront/WAF reduces edge-layer risk and provides a consistent entry point for dashboard, API, and WebSocket traffic.

To prevent CloudFront/WAF bypass, ALB will be configured to forward only valid requests that contain an origin verification header added by CloudFront. The ALB Security Group will also be restricted to suitable sources, such as the CloudFront origin-facing prefix list during implementation. This ensures that the API/backend cannot be accessed directly from the Internet outside the CloudFront-controlled path.

#### 3.4. Backend API and WebSocket

The backend tier will run FastAPI API service and WebSocket service in private subnets. The backend will be exposed through Application Load Balancer and will not be directly opened from the Internet to EC2. A Backend Auto Scaling Group can scale the service based on request rate, CPU, or other metrics suitable for API/WebSocket traffic.

The backend ingestion API will perform authentication, schema validation, normalization, and initial routing. After an event is valid, the backend will send the message to SQS for asynchronous worker processing. The WebSocket service will provide real-time alert feeds for the dashboard, while REST APIs will provide alert list, alert detail, dashboard summary, model status, and data source health.

#### 3.5. SQS and Asynchronous Worker Processing

Amazon SQS will act as the internal queue between the backend ingestion API and the worker tier. The queue absorbs burst traffic from the lab, supports retry, and decouples request ingestion from AI inference. A dead-letter queue will retain failed messages after a defined retry threshold.

The worker tier will include an SQS consumer and AI/Fusion processing worker, running in private subnets without public exposure. A Worker Auto Scaling Group can scale based on queue backlog or message age. Workers will consume messages from SQS, call AI adapters, run the Fusion Layer, write final alerts to RDS/S3, and emit telemetry for monitoring.

#### 3.6. AI/Fusion Layer

The AI layer will include three model groups:

- *AI1 - Anomaly Detection*: detects anomalous network flow behavior from Zeek `conn.log`.
- *AI2A - Network Attack Classification*: classifies behaviors such as port scan, brute force, beaconing, web initial access, or network anomaly from flow-level features.
- *AI2B - HTTP Semantic Web Attack Detection*: detects SQL Injection, Cross-Site Scripting, and semantic web attacks from HTTP method, URI, query string, or payload metadata.

The Fusion Layer will combine model outputs and rule-based evidence into final alerts. Alerts will include attack type, severity, risk score, confidence score, detected_by, MITRE ATT&CK mapping, raw/evidence summary, and decision flow. This allows the dashboard to show explainable final conclusions instead of raw AI labels only.

#### 3.7. RDS/S3 Storage Layer

Amazon RDS for PostgreSQL Multi-AZ deployment will provide a primary DB instance and a standby DB instance in another Availability Zone to improve high availability and support failover. RDS PostgreSQL Multi-AZ will store final alerts, evidence summaries, and dashboard query data. The standby instance is used for high availability and failover, not as a read-serving replica in the default design scope.

Amazon S3 Data Bucket will store raw logs, processed evidence, model artifacts, or exported reports depending on the implementation phase. S3 Gateway Endpoint will be used by backend/worker instances in private subnets to access the S3 Data Bucket, model artifacts, or evidence exports without going through NAT Gateway. Access to the frontend bucket remains handled by CloudFront through Origin Access Control.

#### 3.8. Monitoring, Governance, and Incident Response

CloudWatch will collect logs, metrics, and alarms for backend, workers, queues, RDS, and operational components. CloudTrail will record AWS API activity for audit. SNS will send email/SMS notifications for alarms or important operational events.

IAM will follow least privilege for backend, workers, and deployment roles. Secrets Manager will store DB credentials and API secrets instead of hard-coding them in source code. KMS will support encryption for secrets and sensitive data. Security Groups will restrict network paths between ALB, backend tier, worker tier, RDS, and related services.

#### 3.9. Architectural Decisions and Trade-offs

| Decision | Choice | Reason | Trade-off |
|---|---|---|---|
| Edge entry | CloudFront + WAF | Provides HTTPS delivery, protects public entry, and unifies dashboard/API ingress | Requires careful cache behavior and origin routing |
| Frontend hosting | Private S3 bucket + CloudFront OAC | Keeps the bucket private while CloudFront reads objects through OAC | Requires correct bucket policy configuration |
| API entry | ALB -> private backend EC2 | Separates public entry from private compute, supports health checks and backend scaling | ALB adds hourly and LCU-based cost |
| Ingestion decoupling | SQS Standard + DLQ | Absorbs burst traffic, retries failed messages, and decouples ingestion from AI processing | SQS Standard may deliver a message more than once, requiring idempotency |
| AI processing | Separate worker tier | Scales by queue backlog and avoids blocking API requests | Requires worker lifecycle and DLQ recovery management |
| Final alert storage | RDS PostgreSQL | Fits dashboard query, filtering, alert history, and evidence summaries | Costs more than storing simple JSON on S3 |
| Raw/evidence storage | S3 Data Bucket | Fits logs, artifacts, evidence exports, and low-cost storage | Not optimized for transactional queries |
| Secret management | Secrets Manager + IAM role | Avoids hard-coded DB/API secrets and supports rotation | Requires IAM/KMS control and no screenshots of secret values |

### 4. Technical Implementation
*Implementation Phases*

1. *Local Lab & log generation*: build attacker, pfSense, victim, Zeek sensor, and collector/tailer; generate benign traffic and attack campaigns; collect `conn.log`, `http.log`, and evidence.
2. *Frontend delivery with S3 and CloudFront*: build the React/Vite dashboard, publish static assets to a private S3 bucket, and serve them through CloudFront with Origin Access Control.
3. *Backend API, ALB and private compute*: deploy FastAPI API/WebSocket service in private subnets, expose it through ALB, and configure security groups and health checks.
4. *SQS, worker and asynchronous AI processing*: implement backend ingestion to push valid events to SQS, run workers to consume messages, execute AI/Fusion, and handle retry/DLQ.
5. *RDS/S3 persistence and dashboard query layer*: deploy RDS PostgreSQL, final alert schema, S3 Data Bucket, dashboard query APIs, and evidence/report export.
6. *Monitoring, governance, hardening and cost optimization*: implement CloudWatch, CloudTrail, SNS, IAM least privilege, Secrets Manager, KMS, AWS Budgets, and cost-optimized operating mode.

*Technical Requirements*

- *Backend/API*: FastAPI, WebSocket, REST APIs, schema validation, event normalization, SQS producer, dashboard query APIs.
- *Worker*: SQS consumer, batch processing, AI adapter orchestration, Fusion Layer, RDS/S3 writer, retry/DLQ handling.
- *Frontend*: React/Vite dashboard, real-time alert feed, KPI widgets, attack distribution charts, event detail modal, MITRE mapping, and incident action simulation.
- *AI runtime*: AI1 anomaly detection, AI2A network attack classification, AI2B HTTP semantic web attack detection, confidence/risk scoring, and model artifact management.
- *AWS infrastructure*: CloudFront, WAF, private S3 frontend bucket with OAC, ALB, EC2 Auto Scaling Groups, SQS/DLQ, RDS PostgreSQL Multi-AZ, S3 Data Bucket, CloudWatch, CloudTrail, SNS, IAM, Secrets Manager, KMS.

### 5. Timeline & Milestones
This proposal describes the final target architecture. Implementation will be phased to reduce risk, validate each layer, and control cost.

- *Phase 1 - Local Lab & log generation*: build the lab, generate traffic, collect logs, and define the event contract.
- *Phase 2 - Frontend delivery with S3 and CloudFront*: deploy the static dashboard on a private S3 bucket and serve it through CloudFront/OAC.
- *Phase 3 - Backend API, ALB and private compute*: deploy backend API/WebSocket service in private subnets and expose it through ALB.
- *Phase 4 - SQS, worker and asynchronous AI processing*: add SQS/DLQ, worker tier, AI inference, and Fusion-based asynchronous processing.
- *Phase 5 - RDS/S3 persistence and dashboard query layer*: store final alerts/evidence in RDS/S3, complete dashboard APIs, and support report export.
- *Phase 6 - Monitoring, governance, hardening and cost optimization*: add CloudWatch, CloudTrail, SNS, IAM least privilege, Secrets Manager, KMS, AWS Budgets, and cost optimization.

### 6. Budget Estimation
The cost estimate for the target architecture was created with [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=a306a7de1d836e86cd98fdd1980002edbde0cefb). The [cost estimate file](estimate_cost.pdf) can be downloaded for details.

*Target Architecture Cost Estimate*

The exported estimate records:

- *Upfront cost*: 0.00 USD.
- *Monthly cost*: 283.82 USD/month.
- *Total 12 months cost*: 3,405.84 USD.

This is the estimate for the target architecture, including production-style components such as EC2, RDS PostgreSQL, VPC/NAT Gateway/public IPv4, Application Load Balancer, WAF, CloudWatch, Secrets Manager, KMS, S3, and SQS. It is not the optimized day-to-day lab/demo operating cost.

Main estimate items:

- *Amazon EC2*: 82.85 USD/month.
- *Amazon RDS for PostgreSQL*: 69.03 USD/month.
- *Amazon VPC/NAT/public IPv4*: 101.32 USD/month.
- *Elastic Load Balancing/Application Load Balancer*: 18.46 USD/month.
- *AWS WAF*: 8.03 USD/month.
- *Amazon CloudWatch*: 1.91 USD/month.
- *AWS Secrets Manager*: 0.85 USD/month.
- *AWS Key Management Service (KMS)*: 1.03 USD/month.
- *Amazon S3, Amazon SQS, Amazon CloudFront, Amazon SNS, and AWS CloudTrail*: small or near-zero cost under the current usage assumptions in the estimate.

The numbers reflect usage assumptions at the export date. Services billed by request, data transfer, or log volume, such as S3, SQS, CloudFront, CloudWatch, and WAF, can increase as traffic, log retention, request volume, or stored data increases. Therefore, this estimate is used as a baseline for the target architecture, not a fixed cost commitment.

*Cost-Optimized Implementation Mode*

During the FCAJ implementation and demo phase, the team will operate in cost-optimized mode to control credits. Some target components, such as multi-AZ NAT Gateway, RDS Multi-AZ, backend/worker Auto Scaling, and 24/7 resources, will only be enabled when they need to be tested or demonstrated.

Cost optimization strategies include:

- Run single RDS or Single-AZ when failover testing is not required.
- Reduce or temporarily avoid running multi-AZ NAT Gateway 24/7 during lab work.
- Start backend/worker only for testing or demo instead of running them continuously.
- Enable Auto Scaling, Multi-AZ, or high availability paths only when they need to be validated.
- Use AWS Billing, Cost Explorer, and AWS Budgets to monitor credits, trigger cost alerts, and update the estimate when architecture changes.

*Free Tier/Credit Positioning*

The target architecture is not designed to fit entirely within AWS Free Tier. Free Tier and AWS credits are used only to reduce implementation and demonstration cost during the FCAJ project, not as the main baseline for production-style architecture. AWS Pricing Calculator provides only an estimate; actual cost will depend on usage, traffic, resource runtime, stored data, final configuration, and any applicable taxes or fees.

### 7. Risk Assessment
*Risk Matrix*

- *Cost overrun*: high impact, medium probability due to NAT Gateway, RDS, EC2, ALB, and WAF hourly or usage-based cost.
- *Queue backlog*: high impact, medium probability if attack campaigns generate more events than workers can process.
- *Schema or feature mismatch*: high impact, medium probability if collector, backend, and AI models do not share the same event contract.
- *AI false positives/false negatives*: high impact, medium probability because models may lack enough data or not cover every attack pattern.
- *WAF/ALB misconfiguration*: medium impact, medium probability because it can break API/WebSocket routing.
- *RDS failover or storage issue*: high impact, low to medium probability.
- *Secret or credential leakage*: high impact, low probability if secrets are copied into docs, screenshots, or source code.
- *Dashboard real-time connection loss*: medium impact, medium probability due to WebSocket timeout, routing, or scaling.

*Mitigation Strategies*

- Separate backend tier and worker tier so each scales on the right signal: request/CPU for backend, queue backlog/message age for workers.
- Use SQS and DLQ to absorb burst traffic, retry failed messages, and support root cause investigation.
- Use schema validation and event normalization at the backend ingestion API before sending messages to SQS.
- Use a consistent model output contract so the Fusion Layer can handle completed, not_applicable, not_available, or failed states transparently.
- Use Secrets Manager, KMS, and IAM least privilege to reduce credential risk.
- Use AWS Budgets/Cost Explorer and cost-optimized mode during demo.
- Use CloudWatch, CloudTrail, and SNS for operations monitoring, audit, and alerting.

*Contingency Plans*

- If lab cost rises quickly, switch to single-instance/Single-AZ mode, stop NAT/worker/backend when not demoing, and enable high availability paths only during validation windows.
- If worker backlog increases, temporarily increase worker capacity or slow down replay/collector rate.
- If WebSocket has issues through CloudFront/ALB, fall back to REST polling for the dashboard during demo.
- If an AI model is not ready, the Fusion Layer can still process rule-based evidence or a model subset with explicit status.

### 8. Expected Outcomes
*Technical Improvements*

The expected result is a real-time SOC dashboard that can receive events from the Local Security Lab, process them through the backend ingestion API, queue work asynchronously with SQS, run multi-model AI inference, combine results through the Fusion Layer, and store final alerts in RDS/S3. The dashboard will display KPIs, alert feeds, evidence, decision flow, MITRE mapping, and system status.

*Long-term Value*

The project creates a practical foundation for security monitoring, cloud architecture, AI security analytics, and incident response. The separated backend/worker architecture supports scaling based on real workload signals, RDS/S3 provides durable analysis data, and CloudWatch/CloudTrail/SNS supports operations and audit. Presenting both the target architecture and the cost-optimized implementation mode demonstrates a balanced understanding of reliability, security, scalability, and cost optimization.
