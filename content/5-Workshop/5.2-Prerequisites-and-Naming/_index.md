---
title: "Prerequisites and Naming"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---


## Introduction

Before deploying the **Hybrid SOC on AWS** system, the implementer needs to prepare the correct AWS environment, local tools, and resource naming conventions.

This section helps the workshop be deployed consistently, avoiding common errors such as creating resources in the wrong region, missing IAM permissions, missing local tools, or exposing secrets in documentation.


## AWS Account and Region

The implementer needs an **AWS account** for the workshop and access to the AWS Console.

Services that need create or read permissions include:

- VPC
- EC2
- IAM Role
- S3
- SQS
- RDS
- Secrets Manager
- CloudWatch

The workshop uses the same region for all resources

In this workshop, the region used is:

```text
ap-southeast-1
```

All resources should be created in the same region to avoid endpoint errors, connection errors, and difficulty managing costs.

How to check the region:

```text
AWS Console
-> Top-right bar
-> Region selector
-> Select Asia Pacific (Singapore) - ap-southeast-1
```

The image below confirms that the AWS Console is set to the correct region **Asia Pacific (Singapore) - ap-southeast-1**.

<img src="/images/5-Workshop/5.2-Prerequisites-and-Naming/w-trang-01-aws-region.jpg" alt="AWS Region" style="max-width: 500px !important; width: auto !important; height: auto !important; display: block !important; margin: 1.5rem auto !important;">


## IAM Permission Check

Before configuring the backend, need to check overall IAM permissions.

How to access IAM Dashboard:

```text
AWS Console
-> Search "IAM"
-> IAM Dashboard
-> Users / Roles / Policies
```

The implementer needs permission to create or read the following services:

```text
VPC, EC2, IAM Role, S3, SQS, RDS, Secrets Manager, CloudWatch
```

If you don't have permission to create IAM Role, ask the account owner or admin to create the role first.

Recommended IAM Role for backend:

```text
socai-backend-role
```

This role will be attached to the EC2 Backend so the backend can call S3, SQS, and Secrets Manager without hard-coding AWS access keys in source code or `.env` files.

The image below shows the IAM Dashboard.

![IAM Dashboard](/images/5-Workshop/5.2-Prerequisites-and-Naming/w-trang-02-iam-dashboard.jpg)


## Local Tools

The local machine needs basic tools to work with source code, check AWS CLI, and connect to EC2.

Tools to prepare:

- Git
- Python
- AWS CLI
- SSH client
- Node.js / pnpm if building frontend
- Local Lab if running end-to-end

Check with commands:

```bash
aws --version
python --version
git --version
```

Check results on local machine:

```text
aws-cli/2.35.15
Python 3.12.10
git version 2.54.0.windows.1
```

The image below demonstrates that the local machine has all the basic tools to continue the workshop.

![Local Tools Version](/images/5-Workshop/5.2-Prerequisites-and-Naming/w-trang-03-local-tools-version.jpg)


## Naming Convention

The workshop uses a unified naming convention to easily manage resources in the AWS Console.

| Resource | Recommended Name |
|---|---|
| VPC | `socai-vpc` |
| EC2 Backend | `socai-backend-ec2` |
| IAM Role Backend | `socai-backend-role` |
| Security Group Backend | `socai-backend-sg` |
| S3 Static Bucket | `socai-static-fe-<account>-<region>` |
| S3 Data Bucket | `socai-data-bucket-<account>-<region>` |
| SQS Main Queue | `socai-events-queue` |
| SQS DLQ | `socai-events-dlq` |
| RDS | `socai-postgres` |

This convention helps the team easily find resources, hand over information, and avoid confusion between components like EC2 Backend, S3 bucket, SQS queue, and RDS database.


##  Convention

Do not put real values or real secrets in the workshop. Values that change by environment should be written as s.

Main s:

```text
<AWS_REGION>
<EC2_PUBLIC_DNS>
<SQS_QUEUE_URL>
<S3_DATA_BUCKET>
<RDS_SECRET_ID>
<RDS_ENDPOINT>
```

Example test backend health endpoint:

```bash
curl -i http://<EC2_PUBLIC_DNS>:8000/health
```

Example environment variables for backend:

```text
AWS_REGION=<AWS_REGION>
SQS_QUEUE_URL=<SQS_QUEUE_URL>
S3_DATA_BUCKET=<S3_DATA_BUCKET>
RDS_SECRET_ID=<RDS_SECRET_ID>
```


## Secret Safety

Do not put sensitive information in markdown, commands, or screenshots.

Do not put the following information in the workshop:

- AWS access key
- AWS secret key
- DB password
- Private key `.pem`
- Token or API key
- Secret value
- Un `.env` content
- Full account ID if not needed

If you need to illustrate configuration, only use  form:

```text
DATABASE_URL=<>
SQS_QUEUE_URL=<>
S3_DATA_BUCKET=socai-data-bucket-...
RDS_SECRET_ID=<>
```


## Conclusion

After this section, the implementer should ensure:

- Has an AWS account for the workshop.
- Has selected the correct region `ap-southeast-1`.
- Has permission to check or create the necessary services.
- Local machine has Git, Python, AWS CLI, and SSH client.
- Resources are named according to the same convention.
- Do not put real secrets in the workshop.

The next section will configure **Networking, IAM, and Security Baseline** for the Backend EC2.
