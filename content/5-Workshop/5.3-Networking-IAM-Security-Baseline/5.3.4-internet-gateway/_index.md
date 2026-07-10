---
title: "Internet Gateway"
date: 2026-07-06
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---


## Introduction

Internet Gateway allows resources in **public subnets** to connect to the Internet. In the **Hybrid SOC on AWS** system, Internet Gateway is often used for public subnets containing ALB, NAT Gateway, or public EC2 in the demo phase.

After creating the VPC, subnets, and route tables, we need to check if the Internet Gateway has been correctly attached to VPC `socai-dev-vpc`.


## Step 1: Go to Internet Gateways page

On the AWS Console, open the **VPC** service, then select **Internet gateways** from the left menu.

If there is no Internet Gateway for the project yet, select **Create internet gateway**.

![Create Internet Gateway](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-04-01-create-internet-gateway.jpg)


## Step 2: Create Internet Gateway

On the Internet Gateway creation screen, enter the name:

![Internet Gateway Settings](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-04-02-internet-gateway-settings.jpg)

Then select **Create internet gateway**.


## Step 3: Attach Internet Gateway to VPC

After creating it, select the newly created Internet Gateway and attach it to the project VPC:

```text
VPC name: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
```

Internet Gateway only works when attached to a VPC.

---

## Step 4: Check Internet Gateway after attaching

After attaching, check that the Internet Gateway is in an attached state with VPC `socai-dev-vpc`.

Information to check:

```text
Internet Gateway name: socai-dev-igw
State: Attached
VPC: socai-dev-vpc
VPC ID: vpc-04c78e55ae4ceec8a
```

![Internet Gateway Attached](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-04-03-internet-gateway-attached.jpg)


## Role in the workshop

Internet Gateway is used for public subnets. The public route table will point the Internet route to the Internet Gateway.

Required route in the public route table:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Public subnets can be used for:

- ALB if configured in a later phase.
- NAT Gateway if private subnets need outbound Internet.
- Public EC2 if a quick demo is needed.


## Conclusion

After this step, the implementer should ensure:

- Internet Gateway has been created.
- Internet Gateway has been attached to VPC `socai-dev-vpc`.
- Public route table can point route `0.0.0.0/0` to Internet Gateway.
- Internet Gateway only serves public subnets.
- If there is a public backend, it's only in demo mode, not a production-style configuration.

The next section will check **NAT Gateway**.
