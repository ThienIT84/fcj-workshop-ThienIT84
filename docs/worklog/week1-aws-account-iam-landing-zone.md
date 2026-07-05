# Week 1 Draft - AWS Account Security & IAM Access Baseline

This file is the working source and implementation guide for Week 1 before publishing to Hugo.

Target Hugo page:

```text
content/1-Worklog/1.1-Week1/_index.md
```

Raw private evidence:

```text
static/images/week1/
```

Public-safe evidence folder:

```text
static/images/worklog/week1/
```

Do not embed raw screenshots from `static/images/week1/` in Hugo. Those screenshots may contain account IDs, credit IDs, user identifiers, or other sensitive information. Only public-safe cropped screenshots should be copied to `static/images/worklog/week1/`.

## Positioning

Use the title:

```text
Week 1 - AWS Account Security & IAM Access Baseline
```

Avoid presenting this week as a full AWS landing zone implementation. In official AWS usage, a landing zone often implies a multi-account environment with governance, organizational units, centralized logging, and AWS Control Tower.

If the phrase **lightweight landing zone** is used, define it clearly:

```text
In this worklog, "lightweight landing zone" refers to a single-account security, identity, Region, and cost baseline. It does not refer to a multi-account AWS Control Tower landing zone.
```

## Publication Mode

The Hugo page should stay in draft mode until the required evidence and time logs are finalized:

```yaml
draft: true
```

Switch to `draft: false` only after:

- Daily worklog dates and time-spent values are finalized.
- Four to six public-safe cropped screenshots are available under `static/images/worklog/week1/`.
- The Hugo page has no placeholder text.
- The Hugo page does not claim validation work that was not performed.

## Evidence Mode

Week 1 uses **minimal evidence mode**.

The public page should prove the main implemented outcomes without turning the worklog into a screenshot album. Credential reports and representative access tests are optional future validation items, not Week 1 public requirements.

Public evidence filenames:

```text
static/images/worklog/week1/
├── w1-e01-root-security.png
├── w1-e02-admin-identity.png
├── w1-e03-group-structure.png
├── w1-e04-viewonly-baseline.png
├── w1-e05-team-users.png
└── w1-e06-membership.png
```

Public evidence meaning:

| ID | Evidence | Proves |
| --- | --- | --- |
| W1-E01 | Root security baseline | Root MFA and root access-key status were reviewed |
| W1-E02 | Administrative IAM identity | `thien-admin` and admin group support daily administration without root |
| W1-E03 | IAM group structure | Role-based groups exist for admin, base users, cloud, backend, AI, and frontend |
| W1-E04 | View-only permission baseline | `FCAJ-Base-Users` receives AWS-managed `ViewOnlyAccess` |
| W1-E05 | Team IAM users | Project members have individual IAM identities |
| W1-E06 | Team users and group assignments | Users are assigned through groups rather than direct permissions |

## Final Hugo Structure

Use this order for the Week 1 public page:

```text
1. Week at a Glance
2. Objective
3. Daily Worklog
4. Technical Implementation Summary
5. IAM Structure
6. Key Validation and Evidence
7. Issues and Troubleshooting
8. Lessons Learned
9. Weekly Outcome
10. Next Week Plan
```

This keeps the page aligned with the FCAJ worklog expectation: daily work, result, issues, and next plan. Screenshots support the worklog but do not replace it.

## Week at a Glance Template

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

## Daily Worklog Template

Use one `Date` value and one `Time spent` value for every daily worklog row.

Only keep days that were actually worked. If Week 1 was implemented in three days, use three rows instead of forcing a five-day log.

Before publishing, verify that the time-spent values match the actual work record. Do not inflate hours just to make the log look heavier.

| Day | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| Day 1 | 17/04/2026 | 2 hours | Reviewed root account security, root MFA status, root access-key status, billing credits, and primary Region selection | Root protection and account constraints were understood before creating project resources | Root should not be used for daily work; `ap-southeast-1` was selected as the primary Region | Create an administrative IAM identity |
| Day 2 | 18/04/2026 | 2.5 hours | Configured the account alias/IAM sign-in flow, created `thien-admin`, and prepared the `FCAJ-Admins` group | Daily administration can be separated from the root user | Administration should use an IAM identity instead of shared root credentials | Design role-based IAM groups for the team |
| Day 3 | 19/04/2026 | 2 hours | Designed IAM groups, created role builder groups, and configured the account password policy | A group-based access model was prepared for cloud, backend, AI, and frontend responsibilities | Forced-MFA and self-service MFA custom policies were deferred to keep Week 1 practical | Add common learning-stage permissions |
| Day 4 | 20/04/2026 | 1.5 hours | Created `FCAJ-Base-Users` and attached AWS-managed `ViewOnlyAccess` | Members can inspect AWS resources during the learning stage without deployment permissions | `ViewOnlyAccess` is broad view-only learning access, not the final least-privilege model | Create individual member users and assign groups |
| Day 5 | 21/04/2026 | 2 hours | Created member IAM users, assigned group memberships, and avoided access keys for Console-only users | Team access was prepared through individual identities rather than shared credentials | Console access was sufficient for the learning stage, so no long-term access keys were created for member users | Move to Week 2 VPC foundation |

## Image Placement

Place cropped screenshots near the claim they prove:

- After account/root context: `w1-e01-root-security.png`
- After admin identity explanation: `w1-e02-admin-identity.png`
- Under IAM Users: `w1-e05-team-users.png`
- Under IAM Groups: `w1-e03-group-structure.png`
- After `ViewOnlyAccess` explanation: `w1-e04-viewonly-baseline.png`
- In evidence section: `w1-e06-membership.png`

Each image should use a short caption. Do not place a long screenshot gallery at the end of the page.

## Technical Implementation Summary

Keep this section short. It should summarize the end state, not repeat every screenshot.

Final Week 1 access baseline:

- Root user is protected and reserved for account-level, recovery, and emergency tasks.
- `thien-admin` is used for routine account administration.
- `FCAJ-Admins` holds administrative access for the designated administrator.
- `FCAJ-Base-Users` provides broad view-only learning access through AWS-managed `ViewOnlyAccess`.
- Role builder groups describe cloud, backend, AI, and frontend responsibilities.
- Builder groups do not receive deployment permissions in Week 1.
- Team users receive individual IAM identities instead of shared credentials.
- Console users do not receive access keys unless a later task explicitly requires programmatic access.

Deployment permissions are intentionally deferred. They should be granted only to selected members during the Week 11 AWS MVP migration sprint and reviewed after the deployment is complete.

## IAM Structure

IAM users:

| IAM user | Responsibility |
| --- | --- |
| `thien-admin` | Account administrator and team leader |
| `fcaj-cloud` | Cloud infrastructure and networking |
| `fcaj-backend` | Backend application and deployment |
| `fcaj-ai` | Dataset, AI models, and detection worker |
| `fcaj-frontend` | Frontend deployment, monitoring, and reporting |

IAM groups:

| IAM group | Purpose in Week 1 |
| --- | --- |
| `FCAJ-Admins` | Administrative account operations |
| `FCAJ-Base-Users` | Common learning-stage access with `ViewOnlyAccess` |
| `FCAJ-Cloud-Builders` | Cloud infrastructure role group, no deployment permission yet |
| `FCAJ-Backend-Builders` | Backend role group, no deployment permission yet |
| `FCAJ-AI-Builders` | AI/data role group, no deployment permission yet |
| `FCAJ-Frontend-Builders` | Frontend role group, no deployment permission yet |

## Full Internal Evidence Archive

The older 17-item list can remain as an internal archive map. It is not a public page requirement.

```text
static/images/worklog/week1/
├── 01-root-mfa-enabled.png
├── 02-root-no-access-key.png
├── 03-free-tier-credits.png
├── 04-primary-region-singapore.png
├── 05-account-alias.png
├── 06-thien-admin-created.png
├── 07-fcaj-admins-policy.png
├── 08-thien-admin-mfa.png
├── 09-thien-admin-signed-in.png
├── 10-account-password-policy.png
├── 11-fcaj-base-users-viewonly.png
├── 12-iam-groups-overview.png
├── 13-builder-group-no-deployment-permission.png
├── 14-five-iam-users-list.png
├── 15-team-user-group-memberships.png
├── 16-iam-credential-report.png
└── 17-member-access-validation.png
```

## Optional Future Validation

The following items may improve the report later, but they are not required for the Week 1 public page:

- Redacted IAM credential report with only safe columns.
- Representative read-only access check.
- Representative denied administrative action.

If these are not performed, do not mention them in the Hugo page.

## Issues And Troubleshooting

| Issue / decision | Resolution |
| --- | --- |
| Root credentials should not be shared with team members | Created `thien-admin` and member IAM users for daily access |
| AWS Free Tier credits were active | Kept the account standalone and did not enable AWS Organizations or AWS Control Tower during Week 1 |
| Forced-MFA policy adds complexity during first onboarding | Deferred forced-MFA JSON and self-service MFA custom policy; active users manually enroll MFA during onboarding |
| `ViewOnlyAccess` is broader than a final production least-privilege policy | Used it only as learning-stage read access and documented that it can be narrowed later |
| Builder groups could accidentally imply deployment access | Created the groups for role structure but did not attach deployment permissions in Week 1 |
| Evidence may expose sensitive AWS information | Keep raw screenshots private and publish only selected cropped evidence |

## Publication Checklist

Before changing the Hugo page to `draft: false`, confirm:

- Daily worklog dates and time-spent values are explicit and correct.
- The Week 1 page has no placeholder text.
- Four to six public-safe cropped screenshots are placed under `static/images/worklog/week1/`.
- Raw screenshots from `static/images/week1/` are not embedded.
- The public page uses only the minimal evidence set.
- The public page does not claim credential report or access-test validation unless those tasks were actually performed.
- Week 2 transition points to VPC Network Foundation.

## Next Week Plan

Week 2 will use the secured IAM baseline to build the VPC network foundation:

- VPC CIDR design
- Public and private subnets
- Route tables
- Internet Gateway
- Security Groups
- Network segmentation pattern for backend, AI engine, and database workloads
