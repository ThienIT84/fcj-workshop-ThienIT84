---
title: "Route Tables"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---


## Introduction

Route tables determine where traffic in a subnet will go. In the **Hybrid SOC on AWS** system, route tables are used to separate the paths of public subnets, private app subnets, and private DB subnets.

After creating the VPC and subnets, we need to check the route tables belonging to VPC `socai-dev-vpc`.


## Step 1: Go to Route Tables page

On the AWS Console, open the **VPC** service, then select **Route tables** from the left menu.

Only check route tables belonging to the project VPC:

```text
VPC: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
```

The image below shows the list of route tables in the project VPC.

![Route Tables List](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-03-01-route-tables-list.jpg)


## Step 2: Check existing Route Tables

In the current project, the main route tables include:

| Route table name | Role | Notes |
|---|---|---|
| `socai-dev-rtb-public` | Public route table | Associated with 2 public subnets |
| `socai-dev-rt-private-a` | Private app route table AZ A | Associated with private subnet A |
| `social-dev-rt-private-b` | Private app route table AZ B | Associated with private subnet B |
| `socai-dev-rt-db` | Database route table | Associated with 2 private DB subnets |

There is also the default route table of the VPC. This route table is not the main route table used to describe the public, private app, and database layers in the workshop.

## Step 3: Check subnet association

In the **Explicit subnet associations** column, need to check if the route table is associated with the correct subnet.

According to the current image:

```text
socai-dev-rtb-public -> 2 subnets
socai-dev-rt-db -> 2 subnets
socai-dev-rt-private-a -> private subnet A
social-dev-rt-private-b -> private subnet B
```

Associating the correct subnet helps ensure:

- Public subnet uses the correct public route table.
- Private app subnet uses the correct private route table.
- Private DB subnet uses the correct database route table.
- Database subnet is not mistakenly associated with the public route table.


## Step 4: Check route table status

Need to ensure route tables belong to the correct VPC:

```text
vpc-04c78e55ae4ceec8a
```

Do not use route tables belonging to the default VPC:

```text
vpc-0bc9182791a842901
```

The route table of the default VPC does not belong to the Hybrid SOC project and should not be used in this workshop.


## Role of Route Tables in the workshop

| Route table type | Role |
|---|---|
| Public route table | Used for public subnets, usually routes to Internet Gateway |
| Private app route table | Used for Backend EC2 or Worker in private subnets |
| DB route table | Used for private DB subnets of RDS PostgreSQL |


## Conclusion

After this step, the implementer should ensure:

- Route tables belong to the correct VPC `socai-dev-vpc`.
- Public route table is associated with public subnets.
- Private app route tables are associated with private app subnets.
- DB route table is associated with private DB subnets.
- Route tables belonging to the default VPC are not used for this project.

The next section will check **Internet Gateway**.
