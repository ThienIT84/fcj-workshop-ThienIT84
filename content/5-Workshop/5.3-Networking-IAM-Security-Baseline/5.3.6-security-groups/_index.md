---
title: "Security Groups"
date: 2026-07-08
weight: 6
chapter: false
pre: " <b> 5.3.6. </b> "
---

## Introduction

Security Group acts as a firewall for resources in AWS. In the **Hybrid SOC on AWS** system, Security Group is used to control inbound and outbound traffic for components like ALB, Backend EC2, AI Worker, and RDS PostgreSQL.

After checking VPC, subnets, route tables, Internet Gateway, and NAT Gateway, the next step is to check the Security Groups belonging to VPC `socai-dev-vpc`.

## Step 1: Check Security Groups list

On the AWS Console, open the **EC2** service, then select **Security Groups** under **Network & Security**.

In this workshop, only check Security Groups belonging to the project VPC:

```text
VPC ID: vpc-04c78e55ae4ceec8a
VPC Name: socai-dev-vpc
```

Current Security Groups in the project include:

| Security Group name | Security Group ID | Role |
|---|---|---|
| `socai-dev-alb-sg` | `sg-0e6d896dc7e2f9d56` | Security Group for ALB |
| `socai-dev-ai-worker-sg` | `sg-0c769c2c9935d039f` | Security Group for AI Worker |
| `socai-dev-rds-sg` | `sg-0f8e019c2101c8f08` | Security Group for RDS PostgreSQL |
| `socai-dev-backend-sg` | `sg-0de2d07c6a356954a` | Security Group for Backend EC2 |
| `default` | `sg-045764b4d04e69246` | Default Security Group in project VPC |

Do not use Security Groups belonging to the default VPC:

```text
VPC ID: vpc-0bc9182791a842901
```

The image below shows the Security Groups belonging to VPC `socai-dev-vpc`.

![Security Groups List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-06-01-security-groups-list.jpg)

## Step 2: Check Backend Security Group

Backend EC2 uses Security Group:

```text
Security Group name: socai-dev-backend-sg
Security Group ID: sg-0de2d07c6a356954a
Description: FastAPI backend and WebSocket instances
VPC ID: vpc-04c78e55ae4ceec8a
```

This Security Group is used to control traffic to Backend EC2.

Backend Security Group currently has:

```text
Inbound rules count: 1
Outbound rules count: 1
```

## Step 3: Check Inbound Rules

Inbound rules determine which traffic is allowed to enter Backend EC2.

In the current project, Backend Security Group has 1 inbound rule:

```text
Type: Custom TCP
Protocol: TCP
Port range: 8000
Source: ALB's Security Group
```

Port `8000` is the port used by the FastAPI backend. The source of the rule is another Security Group, not `0.0.0.0/0`, so the backend does not directly expose port `8000` to the entire Internet.

This is consistent with the model:

```text
Client
-> ALB
-> Backend EC2 port 8000
```

The image below shows the inbound rule of `socai-dev-backend-sg`.

![Backend Security Group Inbound](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-06-02-backend-sg-inbound.jpg)

## Step 4: Check Outbound Rules

Outbound rules determine whether Backend EC2 is allowed to call external services or other services.

In the current project, Backend Security Group has 1 outbound rule:

```text
Type: All traffic
Protocol: All
Port range: All
Destination: 0.0.0.0/0
```

This rule allows the backend to send outbound traffic. The backend can use outbound traffic to call necessary services or endpoints like SQS, S3, Secrets Manager, CloudWatch, or RDS if the route table, IAM Role, and destination Security Group allow it.

In the demo environment, the wide outbound like this makes testing easier. In a production environment, you can tighten outbound to the correct destination and port as needed.

The image below shows the outbound rule of `socai-dev-backend-sg`.

![Backend Security Group Outbound](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-06-03-backend-sg-outbound.jpg)

## Role of Security Groups in the project

| Security Group | Role |
|---|---|
| `socai-dev-alb-sg` | Receives traffic from client and forwards to backend |
| `socai-dev-backend-sg` | Allows backend to receive traffic from ALB via port `8000` |
| `socai-dev-ai-worker-sg` | Used for event processing worker |
| `socai-dev-rds-sg` | Protects RDS PostgreSQL |

The Backend Security Group currently does not show SSH port `22` rule. Therefore, in this workshop, do not claim the backend is opening SSH without real evidence.

## Demo mode and ALB mode

In the current architecture, backend inbound is following ALB mode:

```text
Client
-> ALB
-> Backend EC2 port 8000
```

With this configuration, backend port `8000` should only receive traffic from the ALB's Security Group.

Should not directly open backend port `8000` to:

```text
0.0.0.0/0
```

if the backend is already behind ALB.

## Conclusion

After this step, the implementer should ensure:

- Backend EC2 uses Security Group `socai-dev-backend-sg`.
- Backend receives traffic via port `8000`.
- Inbound source is another Security Group, consistent with the ALB model.
- Outbound currently allows `All traffic` to `0.0.0.0/0`.
- Do not use Security Group from default VPC.
- Do not claim SSH port `22` without real rule and screenshot.
- RDS port `5432` should not be public to the Internet.

The next section will check **IAM Role for EC2 Backend**.
