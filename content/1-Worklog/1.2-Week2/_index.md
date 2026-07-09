---
title: "Week 2 - VPC Network Foundation & Segmentation"
date: 2026-05-24
weight: 2
chapter: false
draft: false
pre: " <b> 1.2. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary service | Amazon Virtual Private Cloud |
| Main objective | Build and validate a segmented AWS network foundation |
| Primary Region | `ap-southeast-1` - Asia Pacific (Singapore) |
| VPC CIDR | `10.10.0.0/16` |
| Public subnets | 2 |
| Private subnets | 2 |
| Availability Zones | 2 |
| Internet access | Internet Gateway for public subnet routing |
| Compute deployment | Deferred to Week 3 |
| Operations tooling | Deferred to Week 4 |
| Week status | VPC foundation implemented; time log and one route-association screenshot remain to be finalized |

## Objective

The objective of Week 2 was to design and validate an Amazon VPC network foundation for the FCAJ security monitoring project.

The lab focused on CIDR planning, public/private subnet segmentation, Internet Gateway routing, route-table behavior, and Security Group boundaries. Compute instances, NAT Gateway validation, and operational tooling were intentionally deferred to later weeks so this page could focus on the network architecture itself.

## Project Context

The Hybrid IDS/AI SOC project needs separated network zones for public entry points, backend services, AI processing workers, and future database resources.

The Week 2 VPC lab established the segmentation model that can later place public-facing components in public subnets while keeping backend, AI, and database workloads in private subnets. This was implemented as an AWS networking lab, not as a production migration of the final project.

## Daily Worklog

| Day | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| Day 1 | 24/05/2026 | Confirm before publishing | Reviewed the VPC requirements and planned the IPv4 CIDR layout | VPC address range and subnet ranges were defined | CIDR ranges needed to remain non-overlapping and easy to identify | Create the VPC and subnets |
| Day 2 | 24/05/2026 | Confirm before publishing | Created the VPC and four subnets across two Availability Zones | Public/private subnet layout was established | Subnet names alone do not determine network behavior | Configure the Internet Gateway and public route table |
| Day 3 | 24/05/2026 | Confirm before publishing | Attached the Internet Gateway and configured public routing | Public subnet routing through the Internet Gateway was validated | Internet Gateway attachment alone is not enough without route-table configuration | Review route-table associations |
| Day 4 | 24/05/2026 | Confirm before publishing | Reviewed private routing and Security Group boundaries | Private network zones were separated from direct Internet Gateway routing | Security Groups control traffic permissions but do not make a subnet public or private | Continue with EC2 public/private compute in Week 3 |

## Network Architecture Overview

The VPC was designed with one large private address space and four smaller subnet ranges. Public subnets are reserved for entry points such as a future bastion, load balancer, or dashboard. Private subnets are reserved for internal workloads such as backend services, AI workers, log processing, or databases.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e06-network-architecture.png" alt="Week 2 VPC network architecture">
  <figcaption>Figure 1 - The Week 2 lab separates public routing from private workload zones inside a single VPC.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | VPC CIDR `10.10.0.0/16`, two public subnets, two private subnets, and the Internet Gateway boundary |
| What it proves | The Week 2 design separates public entry zones from private workload zones before compute deployment |
| Limitation | This is a report diagram; the AWS Console screenshots below provide implementation evidence |

## VPC and CIDR Design

The VPC was created with the CIDR range:

```text
10.10.0.0/16
```

This provides a large address space for later compute, database, and service experiments while keeping the lab easy to reason about.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e01-vpc-cidr.png" alt="VPC CIDR configuration">
  <figcaption>Figure 2 - The VPC was created with CIDR <code>10.10.0.0/16</code>; the AWS account owner ID is hidden.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | VPC state is `Available`, default VPC is `No`, and IPv4 CIDR is `10.10.0.0/16` |
| What it proves | A dedicated VPC was created for the lab network instead of using the default VPC |
| Hidden data | Only the AWS account owner ID is hidden; VPC ID and route table ID remain visible |

## Public and Private Subnet Design

The VPC address space was divided into four `/24` subnets:

| Subnet | Type | CIDR | Purpose |
| --- | --- | --- | --- |
| Public Subnet 1 | Public | `10.10.1.0/24` | Future public entry resources |
| Public Subnet 2 | Public | `10.10.2.0/24` | Future public entry resources in another AZ |
| Private Subnet 1 | Private | `10.10.3.0/24` | Future backend, AI, or database workloads |
| Private Subnet 2 | Private | `10.10.4.0/24` | Future backend, AI, or database workloads in another AZ |

Each subnet screenshot is shown separately so the reviewer can read the subnet name, subnet ID, state, VPC, CIDR block, and Availability Zone without losing detail.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02a-public-subnet-1.png" alt="Public Subnet 1">
  <figcaption>Figure 3 - Public Subnet 1 was created as the first public entry subnet with CIDR <code>10.10.1.0/24</code>.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Name `Public Subnet 1`, state `Available`, VPC assignment, CIDR `10.10.1.0/24`, and Availability Zone |
| What it proves | The first public subnet range was created inside the Week 2 VPC |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02b-public-subnet-2.png" alt="Public Subnet 2">
  <figcaption>Figure 4 - Public Subnet 2 was created as a second public entry subnet with CIDR <code>10.10.2.0/24</code>.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Name `Public Subnet 2`, state `Available`, VPC assignment, CIDR `10.10.2.0/24`, and Availability Zone |
| What it proves | A second public subnet was prepared to support multi-AZ style network layout |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02c-private-subnet-1.png" alt="Private Subnet 1">
  <figcaption>Figure 5 - Private Subnet 1 was created for internal workloads with CIDR <code>10.10.3.0/24</code>.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Name `Private Subnet 1`, state `Available`, VPC assignment, CIDR `10.10.3.0/24`, and Availability Zone |
| What it proves | The first private subnet range was reserved for future backend, AI, or database workloads |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02d-private-subnet-2.png" alt="Private Subnet 2">
  <figcaption>Figure 6 - Private Subnet 2 was created for internal workloads with CIDR <code>10.10.4.0/24</code>.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Name `Private Subnet 2`, state `Available`, VPC assignment, CIDR `10.10.4.0/24`, and Availability Zone |
| What it proves | A second private subnet was prepared to support internal workloads in another Availability Zone |

A subnet is classified as public when its associated route table contains a route to an Internet Gateway. For an EC2 instance to communicate with the internet over IPv4, the instance also needs a public IPv4 address or Elastic IP and traffic permissions.

## Internet Gateway and Route Tables

An Internet Gateway was attached to the VPC and used by the public route table.

| Route table | Destination | Target | Meaning |
| --- | --- | --- | --- |
| Public route table | `10.10.0.0/16` | `local` | Internal VPC communication |
| Public route table | `0.0.0.0/0` | Internet Gateway | Internet-bound IPv4 path for public subnets |
| Private route table | `10.10.0.0/16` | `local` | Internal VPC communication |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e03-igw-attached.png" alt="Internet Gateway attached to VPC">
  <figcaption>Figure 7 - The Internet Gateway was attached to the Week 2 VPC; the AWS account owner ID is hidden.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Internet Gateway state is `Attached` and the VPC ID points to the Week 2 VPC |
| What it proves | The VPC has an Internet Gateway available for public route-table targets |
| Hidden data | Only the AWS account owner ID is hidden; Internet Gateway ID and VPC ID remain visible |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e04-public-route-table.png" alt="Public route table">
  <figcaption>Figure 8 - The public route table contains local VPC routing and a default route to the Internet Gateway.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Route `10.10.0.0/16 -> local` and route `0.0.0.0/0 -> Internet Gateway` are both active |
| What it proves | Public subnets can have an internet-bound IPv4 path when associated with this route table |
| Limitation | A separate route-table association screenshot was not captured, so association is documented from lab notes |

Private subnet outbound internet access through NAT Gateway is not part of the Week 2 outcome. It will be documented in Week 3 together with EC2 public/private compute validation.

## Security Group Segmentation

Security Groups were reviewed as resource-level traffic controls for the VPC lab. They do not define whether a subnet is public or private; route tables define the network path, while Security Groups define whether traffic is allowed for attached resources.

The lab-stage rules include broad access used during hands-on validation. This is not the intended production posture. A production version should restrict administrative access to known source IPs, bastion-only paths, or managed private access patterns.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e05a-public-sg.png" alt="Public subnet Security Group">
  <figcaption>Figure 9 - The public subnet Security Group was created for lab-stage SSH and ICMP validation; the AWS account owner ID is hidden.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | Security Group name, VPC ID, inbound SSH rule, inbound ICMP rule, and broad lab-stage source ranges |
| What it proves | Public-facing lab resources were controlled through a dedicated Security Group rather than subnet naming alone |
| Important caveat | Sources such as `0.0.0.0/0` are lab-stage access rules, not the intended production baseline |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e05b-private-sg.png" alt="Private subnet Security Group">
  <figcaption>Figure 10 - The private subnet Security Group rules were reviewed for controlled internal access patterns.</figcaption>
</figure>

| Evidence note | Detail |
| --- | --- |
| What to check | SSH source from the public Security Group, ICMP source from `10.10.0.0/16`, and the lab-stage HTTPS rule |
| What it proves | Private workload access was modeled separately from public subnet access |
| Important caveat | The broad HTTPS source was kept for lab exploration and should be restricted in a production design |

## Validation and Selected Evidence

| ID | Evidence | Result |
| --- | --- | --- |
| W2-E01 | VPC and CIDR `10.10.0.0/16` | Verified |
| W2-E02 | Public Subnet 1, CIDR `10.10.1.0/24` | Verified |
| W2-E03 | Public Subnet 2, CIDR `10.10.2.0/24` | Verified |
| W2-E04 | Private Subnet 1, CIDR `10.10.3.0/24` | Verified |
| W2-E05 | Private Subnet 2, CIDR `10.10.4.0/24` | Verified |
| W2-E06 | Internet Gateway attached to the VPC | Verified |
| W2-E07 | Public route `0.0.0.0/0 -> IGW` | Verified |
| W2-E08 | Public and private Security Group baseline | Verified |
| W2-E09 | Route-table subnet associations | Reviewed in lab notes; dedicated screenshot not captured |

Raw lab notes and raw screenshots are not embedded in the Hugo page. The published page uses selected screenshots under `/images/worklog/week2/`.

## Issues and Troubleshooting

| Issue / decision | Resolution |
| --- | --- |
| Subnet names can be misleading | Documented that route-table association determines public/private behavior |
| Internet Gateway attachment alone does not create usable public routing | Added and validated the public default route to the Internet Gateway |
| Route tables and Security Groups appeared to overlap conceptually | Separated their responsibilities: route tables choose paths, Security Groups allow traffic |
| Integrated lab evidence included EC2, NAT, and operations tooling | Limited Week 2 to VPC foundation and deferred the rest to Week 3 and Week 4 |
| Some screenshots contained AWS account identity information | Hid only account owner IDs while keeping resource IDs visible for evidence value |

## Lessons Learned

Subnet naming is not enough. The actual public or private behavior depends on route tables, subnet associations, public addressing, and traffic permissions.

Another important lesson was that VPC design should be reviewed as a system. CIDR planning, subnet layout, Internet Gateway attachment, route tables, and Security Groups all work together. A mistake in any one of them can make a resource unreachable or accidentally exposed.

## Weekly Outcome

By the end of Week 2, the project had a validated AWS VPC network foundation with a defined CIDR range, four subnets, public routing through an Internet Gateway, private subnet separation, and a Security Group baseline.

This network model is ready for Week 3, where EC2 public/private instances, bastion-style access, NAT Gateway, and private outbound connectivity will be documented.

## Next Week Plan

Week 3 will deploy compute resources into the VPC created in Week 2. The focus will be:

- Public EC2 instance
- Private EC2 instance
- Bastion-style SSH access
- NAT Gateway and Elastic IP
- Private subnet outbound internet validation
- NAT cost awareness and cleanup
