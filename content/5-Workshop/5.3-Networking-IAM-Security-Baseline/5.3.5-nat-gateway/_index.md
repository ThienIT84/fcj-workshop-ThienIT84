---
title: "NAT Gateway"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 5.3.5. </b> "
---

## Introduction

NAT Gateway allows instances in **private subnets** to go outbound to the Internet, for example updating packages, pulling source code, or calling external endpoints.

In the **Hybrid SOC on AWS** system, NAT Gateway is used to support Backend EC2 or Worker in private subnets to access the Internet when needed, but the Internet cannot actively access the private instance in reverse.

## Step 1: Go to NAT Gateways page

On the AWS Console, open the **VPC** service, then select **NAT gateways** from the left menu.

In the current project, NAT Gateway has been created with the name:

```text
socai-dev-nat-a
```

The image below shows that the NAT Gateway is in `Available` state.

![NAT Gateway List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-05-01-nat-gateway-list.jpg)

## Step 2: Check NAT Gateway details

Select NAT Gateway `socai-dev-nat-a` to check detailed information.

Current NAT Gateway information:

```text
NAT Gateway name: socai-dev-nat-a
NAT Gateway ID: nat-0a7e6ff8f16b60be7
Connectivity type: Public
State: Available
VPC: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
Subnet: social-dev-public-subnet-a
Subnet ID: subnet-09ac666babe09a780
Primary Private IPv4: 10.20.2.27
Primary Public IPv4: 47.131.186.17
```

NAT Gateway is located in public subnet `social-dev-public-subnet-a`, this is the correct configuration because NAT Gateway needs a path to the Internet via Internet Gateway.

![NAT Gateway Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-05-02-nat-gateway-detail.jpg)

## Step 3: Identify private subnets using NAT Gateway

The project's private app subnets include:

```text
social-dev-private-subnet-a
social-dev-private-subnet-b
```

These two subnets are used for Backend EC2 or Worker in the private layer.

If instances in private subnets need to access the Internet, the route table of the private subnet needs to have a route through NAT Gateway.

Route to check in the private route table:

```text
Destination: 0.0.0.0/0
Target: nat-0a7e6ff8f16b60be7
```

The image below shows the private app subnets belonging to VPC `socai-dev-vpc`.

![Private App Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-05-03-private-app-subnets.jpg)

## Role of NAT Gateway in the workshop

| Component | Role |
|---|---|
| NAT Gateway | Allows private subnets to go outbound to the Internet |
| Public subnet | Where NAT Gateway is placed |
| Private app subnet | Where Backend EC2 or Worker is placed |
| Internet Gateway | Allows NAT Gateway to go out to the Internet |
| Route table | Directs traffic from private subnets to NAT Gateway |

NAT Gateway helps backend or worker in private subnets access the Internet without needing a direct public IP.

## Conclusion

After this step, the implementer should confirm:

- NAT Gateway `socai-dev-nat-a` has been created.
- NAT Gateway is in `Available` state.
- NAT Gateway belongs to VPC `socai-dev-vpc`.
- NAT Gateway is placed in public subnet `social-dev-public-subnet-a`.
- Private app subnets can use NAT Gateway for outbound Internet if the route table is correctly pointed.
- Private route tables need to be checked to ensure there is a route `0.0.0.0/0` to NAT Gateway.

The next section will check **Security Groups**.
