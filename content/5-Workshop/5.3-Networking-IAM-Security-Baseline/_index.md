---
title: "Networking IAM Security Baseline"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---


## Introduction

This section checks the **network, IAM, and security baseline** so the EC2 Backend can run safely and connect to necessary services like SQS, S3, RDS, Secrets Manager, and CloudWatch.

Contents to check include:

- VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- IAM Role for EC2 Backend

If any service is not configured yet, clearly state **not configured** or **extension phase**, do not claim it is deployed.

## Goals

After this section, the implementer should be able to identify:

- The VPC being used for the project.
- Which subnets are used for public, app, and database layers.
- Which route tables are for public/private subnets.
- Whether a NAT Gateway exists or not.
- Which ports are open on the Backend Security Group.
- Whether the EC2 Backend uses an IAM Role instead of access keys.

## List of lessons in this section

- [VPC Overview](./5.3.1-vpc-overview/)
- [Subnets](./5.3.2-subnets/)
- [Route Tables](./5.3.3-route-tables/)
- [Internet Gateway](./5.3.4-internet-gateway/)
- [NAT Gateway](./5.3.5-nat-gateway/)
- [Security Groups](./5.3.6-security-groups/)
- [IAM Role for EC2 Backend](./5.3.7-iam-role-backend/)
