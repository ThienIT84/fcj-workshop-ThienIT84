---
title: "VPC Overview"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---


## Introduction

VPC is the private network of the **Hybrid SOC on AWS** system. This is the network foundation for deploying Backend EC2, Worker, RDS PostgreSQL, Security Group, and related network components.

In this workshop, we create a dedicated VPC for the project with the name:

```text
socai-dev-vpc
```

This VPC will be used for the next steps like creating subnets, route tables, Internet Gateway, NAT Gateway, and Backend EC2.


## Step 1: Create VPC

On the AWS Console, open the **VPC** service, go to **Your VPCs** page and select **Create VPC** as shown below.

![Create VPC Button](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-01-create-vpc-button.jpg)


## Step 2: Configure VPC

On the **Create VPC** screen, configure as follows:

In this workshop, select **VPC only** to manually configure subsequent components like subnets, route tables, Internet Gateway, and NAT Gateway.

The `10.20.0.0/16` CIDR block is sufficient to be divided into multiple subnets for public layer, app layer, and database layer.

![Create VPC Settings](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-02-create-vpc-settings.jpg)

After verifying the information is correct, select **Create VPC**.


## Step 3: Check VPC after creation

After creation, return to the **Your VPCs** page and check the newly created VPC.

Information to see:

![VPC List Created](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-03-vpc-list-created.jpg)


## Step 4: Check VPC details

Select VPC `socai-dev-vpc` to check detailed information.

Need to confirm key information:


![VPC Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-01-04-vpc-detail.jpg)


## Role of VPC in the workshop

In the Hybrid SOC architecture, the VPC is used to separate the main network layers:

| Network Layer | Role |
|---|---|
| Public subnet | Used for ALB or NAT Gateway if configured |
| Private app subnet | Used for Backend EC2 or Worker |
| Private DB subnet | Used for RDS PostgreSQL |

Creating a dedicated VPC helps the system be easier to manage, avoids sharing the network with other projects, and facilitates controlling route tables, security groups, and database access.


## Notes

At this step, we only create **VPC only**, so the following components are not automatically created:

```text
Subnets
Internet Gateway
NAT Gateway
Custom Route Tables
Backend-specific Security Groups
```

These components will be configured in subsequent steps.

Before adding screenshots to the workshop, need to redact or crop sensitive information like Owner ID or account ID if they appear in the images.


## Conclusion

After this step, the implementer should ensure:

- VPC `socai-dev-vpc` has been created.
- IPv4 CIDR is `10.20.0.0/16`.
- VPC is in `Available` state.
- Tenancy is `default`.
- DNS resolution and DNS hostnames are enabled.
- VPC is ready to continue creating subnets, route tables, and security baseline.

The next section will check **Subnets**.
