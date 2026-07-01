---
title: "Week 1 - AWS Account Security & IAM Access Baseline"
date: 2026-04-17
weight: 1
chapter: false
draft: true
pre: " <b> 1.1. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary service | AWS Identity and Access Management |
| Main objective | Secure the AWS account and create individual team access |
| Key output | Root protection, admin IAM user, IAM groups, and team users |
| Primary Region | `ap-southeast-1` - Asia Pacific (Singapore) |
| Account model | Standalone AWS account |
| Start date | 17/04/2026 |
| Completion date | 21/04/2026 |
| Migration status | Not started |
| Week status | Implementation completed; selected public evidence added |

## Objective

The objective of Week 1 was to establish a secure AWS account and IAM access baseline before creating cloud infrastructure for the FCAJ project.

This week focused on root user protection, billing awareness, Region selection, individual IAM users, group-based permissions, and a practical access model for a small internship team. The account currently remains standalone; AWS Organizations and AWS Control Tower were not enabled during this phase.

In this worklog, **lightweight landing zone** means a single-account security, identity, Region, and cost baseline. It does not mean a multi-account AWS Control Tower landing zone.

![Root security baseline](/images/worklog/week1/w1-e01-root-security.png)

*Figure 1 - Root MFA and root access-key status were reviewed before project resources were created.*

## Daily Worklog

| Day | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| Day 1 | 17/04/2026 | 2 hours | Reviewed root account security, root MFA status, root access-key status, billing credits, and primary Region selection | Root protection and account constraints were understood before creating project resources | Root should not be used for daily work; `ap-southeast-1` was selected as the primary Region | Create an administrative IAM identity |
| Day 2 | 18/04/2026 | 2.5 hours | Configured the account alias/IAM sign-in flow, created `thien-admin`, and prepared the `FCAJ-Admins` group | Daily administration can be separated from the root user | Administration should use an IAM identity instead of shared root credentials | Design role-based IAM groups for the team |
| Day 3 | 19/04/2026 | 2 hours | Designed IAM groups, created role builder groups, and configured the account password policy | A group-based access model was prepared for cloud, backend, AI, and frontend responsibilities | Forced-MFA and self-service MFA custom policies were deferred to keep Week 1 practical | Add common learning-stage permissions |
| Day 4 | 20/04/2026 | 1.5 hours | Created `FCAJ-Base-Users` and attached AWS-managed `ViewOnlyAccess` | Members can inspect AWS resources during the learning stage without deployment permissions | `ViewOnlyAccess` is broad view-only learning access, not the final least-privilege model | Create individual member users and assign groups |
| Day 5 | 21/04/2026 | 2 hours | Created member IAM users, assigned group memberships, and avoided access keys for Console-only users | Team access was prepared through individual identities rather than shared credentials | Console access was sufficient for the learning stage, so no long-term access keys were created for member users | Move to Week 2 VPC foundation |

## Technical Implementation Summary

The Week 1 implementation started from a simple problem: the project had an AWS account and credits, but the team could not safely share root credentials. The work therefore moved from account protection to individual access.

The final access baseline is:

- Root user is protected and reserved for account-level, recovery, and emergency tasks.
- `thien-admin` is used for routine account administration.
- `FCAJ-Admins` holds administrative access for the designated administrator.
- `FCAJ-Base-Users` provides broad view-only learning access through AWS-managed `ViewOnlyAccess`.
- Role builder groups describe cloud, backend, AI, and frontend responsibilities.
- Builder groups do not receive deployment permissions in Week 1.
- Team users receive individual IAM identities instead of shared credentials.
- Console users do not receive access keys unless a later task explicitly requires programmatic access.

![Administrative IAM identity](/images/worklog/week1/w1-e02-admin-identity.png)

*Figure 2 - Routine administration is performed through thien-admin instead of the root user.*

Deployment permissions are intentionally deferred. They should be granted only to selected members during the Week 11 AWS MVP migration sprint and reviewed after the deployment is complete.

![View-only permission baseline](/images/worklog/week1/w1-e04-viewonly-baseline.png)

*Figure 3 - FCAJ-Base-Users receives broad view-only learning access through ViewOnlyAccess.*

## IAM Structure

### IAM Users

| IAM user | Responsibility |
| --- | --- |
| `thien-admin` | Account administrator and team leader |
| `fcaj-cloud` | Cloud infrastructure and networking |
| `fcaj-backend` | Backend application and deployment |
| `fcaj-ai` | Dataset, AI models, and detection worker |
| `fcaj-frontend` | Frontend deployment, monitoring, and reporting |

![Team IAM users](/images/worklog/week1/w1-e05-team-users.png)

*Figure 4 - Project members use individual IAM identities instead of shared credentials.*

### IAM Groups

| IAM group | Purpose in Week 1 |
| --- | --- |
| `FCAJ-Admins` | Administrative account operations |
| `FCAJ-Base-Users` | Common learning-stage access with `ViewOnlyAccess` |
| `FCAJ-Cloud-Builders` | Cloud infrastructure role group, no deployment permission yet |
| `FCAJ-Backend-Builders` | Backend role group, no deployment permission yet |
| `FCAJ-AI-Builders` | AI/data role group, no deployment permission yet |
| `FCAJ-Frontend-Builders` | Frontend role group, no deployment permission yet |

![IAM group structure](/images/worklog/week1/w1-e03-group-structure.png)

*Figure 5 - IAM groups separate administration, common learning access, and project responsibilities.*

Permissions were attached to groups instead of directly to individual users. This makes the account easier to review and keeps the permission model adjustable as the project moves from learning labs to migration work.

## Key Validation and Evidence

Raw screenshots and implementation notes are kept as private evidence. The published report uses selected public-safe cropped screenshots instead of raw console captures.

| ID | Evidence | Result |
| --- | --- | --- |
| W1-E01 | Root security baseline | Verified |
| W1-E02 | Administrative IAM identity | Verified |
| W1-E03 | IAM group structure | Verified |
| W1-E04 | View-only permission baseline | Verified |
| W1-E05 | Team IAM users | Verified |
| W1-E06 | Team users and group assignments | Verified |

![Team group membership](/images/worklog/week1/w1-e06-membership.png)

*Figure 6 - Member users are assigned through IAM groups rather than direct permissions.*

## Issues and Troubleshooting

| Issue / decision | Resolution |
| --- | --- |
| Root credentials should not be shared with team members | Created `thien-admin` and member IAM users for daily access |
| AWS Free Tier credits were active | Kept the account standalone and did not enable AWS Organizations or AWS Control Tower during Week 1 |
| Forced-MFA policy adds complexity during first onboarding | Deferred forced-MFA JSON and self-service MFA custom policy; active users manually enroll MFA during onboarding |
| `ViewOnlyAccess` is broader than a final production least-privilege policy | Used it only as learning-stage read access and documented that it can be narrowed later |
| Builder groups could accidentally imply deployment access | Created the groups for role structure but did not attach deployment permissions in Week 1 |
| Evidence may expose sensitive AWS information | Kept raw screenshots private and published only selected cropped evidence |

## Lessons Learned

AWS security starts before any infrastructure is created. A VPC, EC2 instance, database, or storage bucket should not be created until the account has a clear identity and access model.

Group-based access is easier to review than assigning policies directly to users. It also makes it easier to grant temporary deployment permissions during the Week 11 AWS MVP migration sprint and remove them after validation.

Another lesson is that screenshots are evidence, not the worklog itself. The worklog should explain what was done, what changed, what decisions were made, and what remains to be finalized before publishing.

## Weekly Outcome

By the end of Week 1, the project had a secure AWS account access baseline. The team reserved the root user for account-level, recovery, and emergency tasks, routine administration moved to `thien-admin`, team identities were separated by responsibility, and common learning access was managed through IAM groups.

The account is ready for the next AWS service labs while deployment permissions remain intentionally deferred until the AWS MVP migration sprint.

## Next Week Plan

Week 2 will build the AWS network foundation for the project. The focus will be:

- VPC design
- Public and private subnets
- Route tables
- Internet Gateway
- Security Groups
- Network segmentation for later backend, AI engine, and database workloads

This moves the project from account access security to cloud network design.
