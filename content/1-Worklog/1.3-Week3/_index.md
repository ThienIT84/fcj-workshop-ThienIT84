---
title: "Week 3 - EC2, Bastion Access & NAT Private Egress"
date: 2026-05-03
weight: 3
chapter: false
draft: false
pre: " <b> 1.3. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary service | Amazon EC2 |
| Supporting services | Key Pair, Security Group, Elastic IP, NAT Gateway, Route Table |
| Main objective | Deploy public/private compute and validate bastion-style private access |
| Primary Region | `ap-southeast-1` - Asia Pacific (Singapore) |
| VPC source | Week 2 VPC foundation |
| Public workload pattern | Bastion / jump host |
| Private workload pattern | Backend, AI worker, or log processor |
| Operations tooling | Deferred to Week 4 |
| Week status | Implemented; final publication pending NAT success evidence review |

## Objective

The objective of Week 3 was to deploy compute resources into the network foundation created in Week 2 and validate a basic public/private access pattern.

The lab focused on Amazon EC2, key pairs, Security Groups, bastion-style SSH access, Elastic IP allocation, NAT Gateway routing, and private subnet outbound design. Operations tooling such as Session Manager, EC2 Instance Connect Endpoint, VPC Flow Logs, Reachability Analyzer, and CloudWatch was intentionally deferred to Week 4.

## Project Context

The Hybrid IDS/AI SOC project should not expose backend, AI processing, or log-processing workloads directly to the internet. A safer pattern is to place public entry resources in public subnets and keep important workloads in private subnets.

In this lab, the public EC2 instance represented a bastion or controlled entry point. The private EC2 instance represented a future backend, AI worker, or internal log processor. NAT Gateway routing was reviewed as the outbound path for private resources that need to download packages or call external services without receiving inbound internet traffic.

## Daily Worklog

| Day | Date | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- |
| Day 1 | 03/05/2026 | Reviewed the Week 2 VPC, public subnets, private subnets, and Security Group model | EC2 placement plan was defined | Important workloads should remain in private subnets | Launch public and private EC2 instances |
| Day 2 | 05/05/2026 | Created public and private EC2 instances with key pairs and Security Groups | Public/private compute pattern was prepared | Public EC2 can act as the bastion entry point | Test private instance access |
| Day 3 | 06/05/2026 | Connected from the public EC2 instance to the private EC2 instance over private IP | Bastion-style SSH access was validated | Private EC2 should not require a public IPv4 address | Configure private outbound path |
| Day 4 | 08/05/2026 | Allocated an Elastic IP, created NAT Gateway resources, and reviewed private route tables | Private subnet route to NAT Gateway was configured | NAT Gateway has cost impact and should be cleaned up after lab use | Review private outbound result |
| Day 5 | 10/05/2026 | Reviewed the private outbound test result and documented the unresolved ping behavior | Troubleshooting note was recorded without overstating success | `ping amazon.com` returned packet loss in the available evidence | Capture a successful outbound test if repeated later |

## EC2 Public and Private Compute Design

The Week 3 compute design used two EC2 roles:

| Component | Placement | Purpose |
| --- | --- | --- |
| Public EC2 | Public subnet | SSH entry point and bastion-style jump host |
| Private EC2 | Private subnet | Internal workload pattern for backend, AI worker, or log processor |

The public instance needs a public IPv4 address, public subnet routing, a Security Group that allows approved SSH access, and a valid key pair. The private instance should not be directly reachable from the internet; access should come from the public EC2 instance or a managed private access method.

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e01-ec2-public-private.png" alt="EC2 public instance details">
  <figcaption>Figure 1 - EC2 public-side evidence showing a running instance with public IPv4 and private IPv4 attributes.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Instance state, public IPv4 address, private IPv4 address, and public DNS |
| What it proves | The public EC2 side of the compute pattern was available for access testing |
| Limitation | This screenshot focuses on public EC2 access attributes; private placement is validated through the bastion SSH evidence below |

## Bastion Access Flow

The private instance was accessed from the public EC2 instance using the private IP address of the private instance. This validated the jump-host pattern:

```text
Laptop
  -> Public EC2 / Bastion
  -> Private EC2 over private IP
```

This model is more appropriate than exposing a private workload directly to the internet because it narrows the administrative entry path.

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e02-bastion-private-ssh.png" alt="Bastion SSH into private EC2">
  <figcaption>Figure 2 - SSH from the public EC2 host into the private EC2 instance succeeded over private IP.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | SSH command from `ip-10-10-1-182` to `10.10.4.240`, Amazon Linux login banner, `whoami`, and private hostname |
| What it proves | The public EC2 instance could reach and administer the private EC2 instance through the VPC private network |
| Important note | The later `ping amazon.com` result in the same screenshot is treated as troubleshooting evidence, not NAT success evidence |

## NAT Gateway and Private Egress Design

Private instances often need outbound internet access for package updates or external API calls. They should not receive inbound internet traffic directly. NAT Gateway supports this pattern by allowing private subnet outbound traffic through a public subnet and Internet Gateway path.

The lab prepared this path using:

| Component | Role |
| --- | --- |
| Elastic IP | Public IPv4 address used by NAT Gateway |
| NAT Gateway | Outbound internet path for private subnets |
| Private route table | Sends `0.0.0.0/0` traffic to the NAT Gateway |

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e03-elastic-ip-for-nat.png" alt="Elastic IP allocated for NAT Gateway">
  <figcaption>Figure 3 - An Elastic IP was allocated for the NAT Gateway used by the private egress path.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Elastic IP allocation success message, EIP name, allocated IPv4 address, and allocation ID |
| What it proves | A static public IPv4 resource was prepared for NAT Gateway |
| Cost note | Elastic IP and NAT Gateway resources can create cost if left running after lab work |

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e04-private-route-to-nat.png" alt="Private route table to NAT Gateway">
  <figcaption>Figure 4 - The private route table sends default outbound traffic to the NAT Gateway; the AWS account owner ID is hidden.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Private route table, private subnet association, route `10.10.0.0/16 -> local`, and route `0.0.0.0/0 -> NAT Gateway` |
| What it proves | Private subnet outbound routing was configured through NAT instead of direct Internet Gateway routing |
| Hidden data | Only the AWS account owner ID is hidden; route table IDs, subnet IDs, VPC IDs, and NAT target remain visible |

## Security Decisions

| Decision | Reason |
| --- | --- |
| Keep internal workloads in private subnets | Backend, AI workers, and log processors should not be directly exposed to the internet |
| Use public EC2 as a bastion-style entry point | Centralizes SSH entry instead of opening direct access to private resources |
| Use Security Groups as resource-level controls | Route tables define network paths; Security Groups define allowed traffic |
| Use NAT for private outbound design | Private resources can initiate outbound traffic without receiving inbound public access |
| Defer managed operations tooling to Week 4 | SSM, EIC Endpoint, Flow Logs, Reachability Analyzer, and CloudWatch deserve a separate operations-focused worklog |

## Validation and Selected Evidence

| ID | Evidence | Result |
| --- | --- | --- |
| W3-E01 | Public EC2 instance access attributes | Verified |
| W3-E02 | Bastion SSH from public EC2 to private EC2 | Verified |
| W3-E03 | Elastic IP allocated for NAT Gateway | Verified |
| W3-E04 | Private route table default route to NAT Gateway | Verified |
| W3-E05 | Private outbound internet success | Not claimed from current evidence |

Raw lab notes and raw screenshots are not embedded in the Hugo page. The published page uses selected screenshots under `/images/worklog/week3/`.

## Issues and Troubleshooting

| Issue / decision | Resolution |
| --- | --- |
| Private EC2 should not be exposed directly to the internet | Used the public EC2 instance as a bastion-style access path |
| Private key permissions are strict for SSH | Used key-based SSH and followed the expected private key permission model |
| `ping amazon.com` showed `100% packet loss` in the available terminal evidence | Documented as troubleshooting instead of claiming NAT success; a future validation should use `curl`, `yum`, or another outbound HTTP/DNS test |
| NAT Gateway and Elastic IP can create cost | Treat NAT resources as lab resources and clean them up after validation |
| EC2 evidence includes sensitive operational context | Public screenshots keep technical IDs visible but avoid exposing AWS account owner ID |

## Lessons Learned

A public subnet is not only a place to run workloads. It can also act as a controlled entry zone for administration, bastion access, or future load balancers.

Private compute is a stronger default for backend and AI components because the instance does not need a public IPv4 address. Access should be mediated through controlled paths such as bastion access or managed AWS operations tooling.

NAT Gateway solves a different problem from bastion access. A bastion helps administrators reach private instances; NAT helps private instances initiate outbound internet connections. They should not be treated as the same networking feature.

## Weekly Outcome

By the end of Week 3, the project had a documented EC2 public/private compute pattern connected to the Week 2 VPC foundation. Bastion-style SSH access into a private instance was validated, and NAT Gateway routing was prepared for private subnet outbound access.

The available evidence does not yet prove successful private outbound internet access, so this page records the NAT route configuration and the unresolved ping behavior honestly.

## Next Week Plan

Week 4 will focus on secure operations and observability for the infrastructure:

- AWS Systems Manager Session Manager
- EC2 Instance Connect Endpoint
- VPC Flow Logs
- Reachability Analyzer
- CloudWatch monitoring and alerting
