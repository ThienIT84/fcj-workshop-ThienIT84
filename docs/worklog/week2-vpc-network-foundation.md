# Week 2 Draft - VPC Network Foundation & Segmentation

This file is the working source and implementation guide for Week 2 before publishing to Hugo.

Target Hugo page:

```text
content/1-Worklog/1.2-Week2/_index.md
```

Raw evidence source:

```text
private-evidence/week2/
```

Public evidence folder:

```text
static/images/worklog/week2/
```

The raw folder is ignored by Git. The Hugo page must not embed raw screenshots from `private-evidence/week2/` or any former `static/images/week_02/` path.

## Positioning

Use the title:

```text
Week 2 - VPC Network Foundation & Segmentation
```

Week 2 should focus on the network foundation only:

- Amazon VPC
- IPv4 CIDR planning
- Public and private subnets
- Internet Gateway
- Route tables
- Subnet routing behavior
- Security Group baseline

EC2, NAT Gateway, EIC Endpoint, Session Manager, Flow Logs, Reachability Analyzer, and CloudWatch belong to Week 3 or Week 4.

## Publication Mode

The Hugo page should stay in draft mode until the dates and time-spent values are finalized:

```yaml
draft: true
```

Switch to `draft: false` only after:

- Daily worklog time-spent values are confirmed.
- Public screenshots are reviewed in `static/images/worklog/week2/`.
- The page has no draft-only language.
- The page does not claim screenshots that were not captured.

## Redaction Rules

Only hide AWS account identity information:

- `Owner ID`
- 12-digit AWS Account ID
- Account ID inside ARN or URL, if present
- Email or account alias, if present

Keep visible:

- VPC ID
- Subnet ID
- Route table ID
- Internet Gateway ID
- Security Group ID
- Resource names
- CIDR blocks
- Availability Zones
- Route destinations and targets
- Security Group rules
- Success messages

## Public Evidence Map

| ID | Public image | Source | Notes |
| --- | --- | --- | --- |
| W2-E01 | `w2-e01-vpc-cidr.png` | `w2-e01-vpc-created.png` | Owner ID hidden; VPC ID and CIDR kept |
| W2-E02 | `w2-e02a-public-subnet-1.png` | `w2-e02a-publicsubnet-layout.png.png` | Full screenshot; proves Public Subnet 1 and CIDR `10.10.1.0/24` |
| W2-E03 | `w2-e02b-public-subnet-2.png` | `w2-e02b-publicsubnet2-layout.png` | Full screenshot; proves Public Subnet 2 and CIDR `10.10.2.0/24` |
| W2-E04 | `w2-e02c-private-subnet-1.png` | `w2-e02C-privatesubnet1-layout.png` | Full screenshot; proves Private Subnet 1 and CIDR `10.10.3.0/24` |
| W2-E05 | `w2-e02d-private-subnet-2.png` | `w2-e02d-privatesubnet2-layout.png` | Full screenshot; proves Private Subnet 2 and CIDR `10.10.4.0/24` |
| W2-E06 | `w2-e03-igw-attached.png` | `w2-e03-igw-attached.png` | Owner ID hidden; IGW ID and VPC ID kept |
| W2-E07 | `w2-e04-public-route-table.png` | `w2-e04-public-route-table.png` | Owner ID hidden; route table ID, local route, and IGW route kept |
| W2-E08 | `w2-e05a-public-sg.png` | `w2-e05a-publicsubnetg-security-groups.png` | Owner ID hidden; lab-stage broad rules kept |
| W2-E09 | `w2-e05b-private-sg.png` | `w2-e05c-privatesubnetupdate-security-groups.png` | Full screenshot; private Security Group rule review |
| W2-E10 | `w2-e06-network-architecture.png` | Generated diagram | Clean architecture overview |

Do not merge multiple screenshots into one public evidence image. Each screenshot should be embedded as a separate figure and followed by a short reading note in the Hugo page. The note should explain what to check, what the screenshot proves, and any limitation such as missing route-association screenshots.

## Deferred Evidence

The following existing images should be reused later, not on the Week 2 public page:

| Source image | Target week |
| --- | --- |
| `w2-e06-ec2-public-private.png` | Week 3 |
| `w2-e07-bastion-private-connection.png` | Week 3 |
| `w2-e08a-nat-private-egress.png` | Week 3 |
| `w2-e08b-nat-private-egress.png` | Week 3 |
| `w2-e09-eic-endpoint-private-access.png` | Week 4 |
| `w2-e11-flow-logs-cloudwatch.png` | Week 4 |

## Technical Wording

Use precise subnet wording:

```text
A subnet is classified as public when its associated route table contains a route to an Internet Gateway.
```

Also include:

```text
For an EC2 instance to communicate with the internet over IPv4, it also needs a public IPv4 address or Elastic IP and traffic permissions.
```

Do not claim that the final project workload is already running in the VPC.

## Publication Checklist

Before changing the Hugo page to `draft: false`, confirm:

- Daily time-spent values are real.
- No raw `static/images/week_02/` paths are linked.
- Public images use `/images/worklog/week2/`.
- Account ID / Owner ID is hidden in every public screenshot.
- The missing route-association screenshot is either provided or honestly marked as not captured.
- Week 3 transition points to EC2 public/private, bastion access, and NAT Gateway.
