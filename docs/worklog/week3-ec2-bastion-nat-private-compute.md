# Week 3 Draft - EC2, Bastion Access & NAT Private Egress

This file is the working source and implementation guide for Week 3 before publishing to Hugo.

Target Hugo page:

```text
content/1-Worklog/1.3-Week3/_index.md
```

Raw evidence source:

```text
private-evidence/week2/images/
```

Public evidence folder:

```text
static/images/worklog/week3/
```

The Hugo page must not embed raw screenshots from `private-evidence/` or any former `static/images/week_02/` path.

## Positioning

Use the title:

```text
Week 3 - EC2, Bastion Access & NAT Private Egress
```

Week 3 should focus on the compute and private access pattern:

- Amazon EC2 public/private placement
- Key pair based SSH
- Public EC2 as bastion-style jump host
- Private EC2 access over private IP
- Elastic IP for NAT Gateway
- Private route table default route to NAT Gateway
- NAT troubleshooting and cost awareness

Do not include Session Manager, EC2 Instance Connect Endpoint, VPC Flow Logs, Reachability Analyzer, or CloudWatch in Week 3. Those belong to Week 4.

## Publication Mode

The Hugo page should stay in draft mode until the dates and time-spent values are finalized:

```yaml
draft: true
```

Switch to `draft: false` only after:

- Daily worklog time-spent values are confirmed.
- Public screenshots are reviewed in `static/images/worklog/week3/`.
- The page has no draft-only language.
- A private outbound success screenshot is added, or the page continues to state that outbound success is not claimed.

## Redaction Rules

Only hide AWS account identity information:

- `Owner ID`
- 12-digit AWS Account ID
- Account ID inside ARN or URL, if present
- Email or account alias, if present

Keep visible:

- EC2 instance IDs
- NAT Gateway ID
- Route table IDs
- Subnet IDs
- VPC IDs
- Private IP addresses
- CIDR blocks
- Route destinations and targets
- Command output
- Success and troubleshooting messages

## Public Evidence Map

| ID | Public image | Source | Notes |
| --- | --- | --- | --- |
| W3-E01 | `w3-e01-ec2-public-private.png` | `w2-e06-ec2-public-private.png` | Public EC2 access attributes; does not replace private subnet placement evidence |
| W3-E02 | `w3-e02-bastion-private-ssh.png` | `w2-e07-bastion-private-connection.png` | Proves bastion SSH into private EC2; ping failure is troubleshooting only |
| W3-E03 | `w3-e03-elastic-ip-for-nat.png` | `w2-e08a-nat-private-egress.png` | Elastic IP allocated for NAT Gateway |
| W3-E04 | `w3-e04-private-route-to-nat.png` | `w2-e08b-nat-private-egress.png` | Owner ID hidden; private route table default route to NAT Gateway remains visible |

Do not merge multiple screenshots into one public evidence image. Each screenshot should be embedded as a separate figure and followed by a short reading note in the Hugo page.

## Evidence Interpretation

Current evidence supports these claims:

- Public EC2 was available for access testing.
- Public EC2 could SSH into the private EC2 instance over private IP.
- Elastic IP was allocated for NAT Gateway.
- Private route table had a default route to NAT Gateway.

Current evidence does not support this claim yet:

```text
Private EC2 outbound internet succeeded.
```

The available terminal screenshot shows `ping amazon.com` with packet loss. Keep that as troubleshooting unless a successful outbound validation screenshot is provided.

Suggested future validation commands:

```bash
curl -I https://amazon.com
curl https://checkip.amazonaws.com
sudo yum makecache
```

## Deferred Week 4 Evidence

The following existing images and notes should be reused later, not on the Week 3 public page:

| Source image or note | Target week |
| --- | --- |
| `w2-e09-eic-endpoint-private-access.png` | Week 4 |
| `w2-e11-flow-logs-cloudwatch.png` | Week 4 |
| `AWS Systems Manager Session Manager.md` | Week 4 |
| `Lab 4.5 EC2 Instance Connect Endpoint (EIC Endpoint).md` | Week 4 |
| `Enable VPC Flow Logs.md` | Week 4 |
| `Using Reachability Analyzer.md` | Week 4 |
| `CloudWatch Monitoring & Alerting.md` | Week 4 |

## Publication Checklist

Before changing the Hugo page to `draft: false`, confirm:

- Daily time-spent values are real.
- No raw `static/images/week_02/` paths are linked.
- No `private-evidence/` path is linked.
- Public images use `/images/worklog/week3/`.
- Account ID / Owner ID is hidden in every public screenshot.
- The page does not claim NAT outbound success unless success evidence exists.
- Week 4 transition points to operations and observability.
