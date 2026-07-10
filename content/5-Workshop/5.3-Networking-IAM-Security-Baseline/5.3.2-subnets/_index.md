---
title: "Subnets"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---


## Introduction

Subnets are used to divide the VPC into multiple small network zones by role. In the **Hybrid SOC on AWS** system, subnets help separate the public layer, backend app layer, and database layer.

After creating VPC `socai-dev-vpc`, we will create and check the subnets belonging to this VPC to prepare for Backend EC2, NAT Gateway or ALB, and RDS PostgreSQL.


## Step 1: Go to Subnets page

On the AWS Console, open the **VPC** service, select **Subnets** from the left menu.

Then select **Create subnet** to start creating a new subnet.

![Create Subnet Button](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-01-create-subnet-button.jpg)


## Step 2: Select VPC for the subnet

In the **VPC ID** section, select the project VPC:

```text
socai-dev-vpc
```

VPC information:

```text
VPC ID: vpc-04c78e55ae4ceec8a
IPv4 CIDR: 10.20.0.0/16
```

All subnets in this section must be created in the same VPC `socai-dev-vpc`.

Do not use subnets with CIDR `172.31.x.x` because those are subnets of the default VPC, not the project VPC.

![Select VPC for Subnet](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-02-setting-subnet.jpg)


## Step 3: Check Public Subnets

Public subnets are used for components that need a public entrypoint like ALB or NAT Gateway.

In the current project, public subnets include:

```text
Subnet name: social-dev-public-subnet-a
Subnet ID: subnet-09ac666babe09a780
IPv4 CIDR: 10.20.0.0/20
```

```text
Subnet name: social-dev-public-subnet-b
Subnet ID: subnet-0f1b248ff01c3b297
IPv4 CIDR: 10.20.16.0/20
```

Public subnets will usually be associated with a route table that has a route to the Internet Gateway:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

![Public Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-03-public-subnet-settings.jpg)


## Step 4: Check Private App Subnets

Private app subnets are used for Backend EC2 or Worker in a production-style architecture.

In the current project, private app subnets include:

```text
Subnet name: social-dev-private-subnet-a
Subnet ID: subnet-0c510623f443f178e
IPv4 CIDR: 10.20.128.0/20
```

```text
Subnet name: social-dev-private-subnet-b
Subnet ID: subnet-071ded17acfbe50a0
IPv4 CIDR: 10.20.144.0/20
```

Backend EC2 in private subnets should not receive traffic directly from the Internet. If public access is needed, requests should go through ALB or CloudFront depending on the final architecture.

![Private App Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-04-private-app-subnet-settings.jpg)


## Step 5: Check Private DB Subnets

Private DB subnets are used for RDS PostgreSQL. The database should not be placed in a public subnet or have a direct route to the Internet Gateway.

In the current project, private DB subnets include:

```text
Subnet name: socai-dev-private-db-subnet-a
Subnet ID: subnet-08fddc2ead9510633
IPv4 CIDR: 10.20.32.0/24
```

```text
Subnet name: socai-dev-private-db-subnet-b
Subnet ID: subnet-06a93d15ec51c7943
IPv4 CIDR: 10.20.33.0/24
```

RDS usually needs a DB subnet group consisting of at least two subnets in two different Availability Zones. However, if the project is demonstrating Single-AZ to reduce costs, record the actual state correctly and do not claim Multi-AZ if it is not enabled.

![Private DB Subnets](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-05-private-db-subnet-settings.jpg)


## Step 6: Check Subnet List

After creating subnets, return to the **Subnets** page and check the list of subnets belonging to the project VPC.

In this workshop, only use subnets belonging to the VPC:

```text
VPC ID: vpc-04c78e55ae4ceec8a
VPC CIDR: 10.20.0.0/16
```

Current project subnet list:

| Subnet name | Role | Subnet ID | IPv4 CIDR |
|---|---|---|---|
| `social-dev-public-subnet-a` | Public subnet | `subnet-09ac666babe09a780` | `10.20.0.0/20` |
| `social-dev-public-subnet-b` | Public subnet | `subnet-0f1b248ff01c3b297` | `10.20.16.0/20` |
| `socai-dev-private-db-subnet-a` | Private DB subnet | `subnet-08fddc2ead9510633` | `10.20.32.0/24` |
| `socai-dev-private-db-subnet-b` | Private DB subnet | `subnet-06a93d15ec51c7943` | `10.20.33.0/24` |
| `social-dev-private-subnet-a` | Private app subnet | `subnet-0c510623f443f178e` | `10.20.128.0/20` |
| `social-dev-private-subnet-b` | Private app subnet | `subnet-071ded17acfbe50a0` | `10.20.144.0/20` |

All the above subnets are in the state:

```text
Available
```

Subnets with CIDR `172.31.x.x` belong to the default VPC and should not be used in this project.

![Subnet List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-06-subnet-list.jpg)


## Step 7: Check Private App Subnet Details

Select a private app subnet to check detailed information.

Example private app subnet being checked:

```text
Subnet name: social-dev-private-subnet-a
Subnet ID: subnet-0c510623f443f178e
IPv4 CIDR: 10.20.128.0/20
Availability Zone: apse1-az2 (ap-southeast-1a)
State: Available
VPC: socai-dev-vpc
Auto-assign public IPv4 address: No
Route table: socai-dev-rt-private-a
```

Private app subnets are used for Backend EC2 or Worker. In a production-style architecture, this subnet should not be directly exposed to the Internet.

![Private App Subnet Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-07-private-app-subnet-detail.jpg)


## Step 8: Check Private DB Subnet Details

Select a private DB subnet to check detailed information.

Example private DB subnet being checked:

```text
Subnet name: socai-dev-private-db-subnet-a
Subnet ID: subnet-08fddc2ead9510633
IPv4 CIDR: 10.20.32.0/24
Availability Zone: apse1-az2 (ap-southeast-1a)
State: Available
VPC: socai-dev-vpc
Auto-assign public IPv4 address: No
Route table: socai-dev-rt-db
```

Private DB subnets are used for RDS PostgreSQL. Database subnets should be kept private to avoid public exposure.

![DB Subnet Detail](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-02-08-db-subnet-detail.jpg)


## Role of subnets in the workshop

| Subnet type | Role |
|---|---|
| Public subnet | Used for ALB or NAT Gateway if configured later |
| Private app subnet | Used for Backend EC2 or Worker |
| Private DB subnet | Used for RDS PostgreSQL |

Dividing subnets by role helps the system be easier to manage, while also reducing the risk of the backend or database being unintentionally exposed publicly.



## Conclusion

After this step, the implementer should ensure:

- All subnets belong to the correct VPC `socai-dev-vpc`.
- Only use subnets with CIDR in the `10.20.0.0/16` range.
- Public subnets are used for ALB or NAT Gateway if available.
- Private app subnets are used for Backend EC2 or Worker.
- Private DB subnets are used for RDS PostgreSQL.
- All subnets are in `Available` state.
- Subnets with `172.31.x.x` belong to the default VPC and should not be used in the project.

The next section will check **Route Tables**.
