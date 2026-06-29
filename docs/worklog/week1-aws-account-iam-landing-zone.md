# Week 1 Draft - AWS Account, IAM & FCAJ Landing Zone

This is the working draft for Week 1 before publishing to Hugo.

Target Hugo page:

```text
content/1-Worklog/1.1-Week1/_index.md
```

Recommended evidence folder:

```text
static/images/worklog/week1/
```

Primary source notes:

```text
/mnt/d/StorgeVault/Thiện học AI/FCAJ-Internship/00_Worklog/week_01/
```

## Objective

Establish a secure AWS account foundation for the FCAJ project before creating infrastructure resources.

The week focuses on AWS account security, IAM access management, billing/credit awareness, and a team permission baseline. This is the landing zone for later VPC, EC2, S3, RDS, CloudWatch, and migration work.

## FCAJ Context

The project uses one standalone AWS account with active AWS Free Tier credits. AWS Organizations will not be enabled in this phase to avoid changing the current credit/account setup.

The project needs multiple team members to access AWS safely, but root credentials must not be shared or used for daily work.

Landing zone principle:

```text
Root user
-> emergency/account-level tasks only

thien-admin
-> account administration and setup

role-based IAM users/groups
-> project members work with only the permissions needed for each phase
```

## Services and Concepts

- AWS Identity and Access Management
- AWS root user security
- IAM users
- IAM user groups
- IAM policies
- AWS managed policies
- Customer-managed policies
- Multi-Factor Authentication
- Account password policy
- Temporary passwords
- Least privilege
- Role-based access control
- Account alias and IAM sign-in URL
- AWS Region selection
- Billing, Free Tier, and credit monitoring

## Planned IAM Structure

IAM users:

| IAM user | Responsibility |
| --- | --- |
| `thien-admin` | Account administrator and team leader |
| `fcaj-cloud` | Cloud infrastructure and networking |
| `fcaj-backend` | FastAPI backend and deployment |
| `fcaj-ai` | Dataset, AI models, and detection worker |
| `fcaj-frontend` | Frontend deployment, monitoring, and reporting |

IAM groups:

| IAM group | Purpose |
| --- | --- |
| `FCAJ-Admins` | Account administration |
| `FCAJ-Base-Users` | Common password/MFA baseline |
| `FCAJ-Cloud-Builders` | VPC, subnet, route table, SG permissions added in Week 2 |
| `FCAJ-Backend-Builders` | EC2/backend deployment permissions added later |
| `FCAJ-AI-Builders` | Dataset/model/worker permissions added later |
| `FCAJ-Frontend-Builders` | S3/CloudFront/frontend permissions added later |

Membership plan:

```text
thien-admin
├── FCAJ-Base-Users
└── FCAJ-Admins

fcaj-cloud
├── FCAJ-Base-Users
└── FCAJ-Cloud-Builders

fcaj-backend
├── FCAJ-Base-Users
└── FCAJ-Backend-Builders

fcaj-ai
├── FCAJ-Base-Users
└── FCAJ-AI-Builders

fcaj-frontend
├── FCAJ-Base-Users
└── FCAJ-Frontend-Builders
```

## Implementation Plan

### Phase 1A - Root User Security

Tasks:

- Sign in using the root user only for bootstrap tasks.
- Verify root MFA status.
- Enable MFA if not already enabled.
- Confirm that root access keys do not exist.
- Reserve root for account recovery, billing, and root-only tasks.

Evidence:

- `01-root-mfa-enabled.png`
- `02-root-no-access-key.png`

Reviewer value:

```text
Shows that the project started with security hygiene before deploying resources.
```

### Phase 1B - Billing Credits and Primary Region

Tasks:

- Review AWS Billing and Credits.
- Record remaining Free Tier credits and expiration date without exposing sensitive account information.
- Select `ap-southeast-1` as the primary working Region.
- Decide not to enable AWS Organizations during this phase due to active credits.

Evidence:

- `03-free-tier-credits.png`
- `04-primary-region-singapore.png`

Reviewer value:

```text
Shows cost awareness and Region consistency before creating lab resources.
```

### Phase 2 - Administrative IAM User

Tasks:

- Create account alias if useful, such as `fcaj-soc-team`.
- Create `FCAJ-Admins`.
- Create `thien-admin`.
- Enable AWS Management Console access.
- Use temporary password and require password change at first sign-in.
- Attach `AdministratorAccess` to `FCAJ-Admins`, not to all users.
- Sign out from root.
- Sign in using `thien-admin`.
- Enable MFA for `thien-admin`.

Evidence:

- `05-account-alias.png`
- `06-fcaj-admins-group.png`
- `07-thien-admin-created.png`
- `08-thien-admin-mfa.png`
- `09-thien-admin-signed-in.png`

Reviewer value:

```text
Shows separation between root and day-to-day administration.
```

### Phase 3 - IAM Groups and Baseline Policies

Tasks:

- Configure account password policy.
- Create `FCAJ-Base-Users`.
- Create builder groups for cloud, backend, AI, and frontend roles.
- Create or document customer-managed policies for self password/MFA management.
- Keep builder groups limited until their service week begins.

Evidence:

- `10-account-password-policy.png`
- `11-fcaj-base-users-created.png`
- `12-iam-groups-overview.png`
- `13-builder-groups-no-service-policy.png`
- `14-self-manage-password-mfa-policy.png`

Reviewer value:

```text
Shows least privilege by delaying service permissions until the relevant project phase.
```

### Phase 4 - Team IAM Users

Tasks:

- Create `fcaj-cloud`.
- Create `fcaj-backend`.
- Create `fcaj-ai`.
- Create `fcaj-frontend`.
- Enable console access with temporary passwords.
- Require password reset at first sign-in.
- Add every user to `FCAJ-Base-Users`.
- Add each user to the correct builder group.
- Do not create access keys unless there is a verified technical requirement.

Evidence:

- `15-iam-users-list.png`
- `16-user-group-membership-cloud.png`
- `17-user-group-membership-backend-ai-frontend.png`
- `18-no-access-keys-for-members.png`

Reviewer value:

```text
Shows individual accountability and avoids shared AWS credentials.
```

### Phase 5 - Validation

Tasks:

- Confirm that each active IAM user can sign in.
- Confirm temporary password change behavior.
- Confirm MFA enrollment.
- Test one allowed operation.
- Test one denied operation, such as attempting IAM changes from a non-admin user.

Evidence:

- `19-member-sign-in-success.png`
- `20-member-mfa-status.png`
- `21-allowed-read-operation.png`
- `22-expected-access-denied.png`

Reviewer value:

```text
Shows that IAM design was validated, not only configured.
```

## Permission Expansion Plan

Permissions should expand with the project, not all at once.

| Week | Permission area |
| --- | --- |
| Week 1 | Identity, sign-in, MFA, baseline read access |
| Week 2 | VPC, subnet, route table, IGW, SG permissions |
| Week 3 | EC2, NAT Gateway, EIP, bastion/private compute permissions |
| Week 4 | SSM, EIC Endpoint, VPC Flow Logs, CloudWatch permissions |
| Week 5 | S3 and CloudFront permissions |
| Week 6 | RDS permissions |
| Week 11 | Backend deployment and artifact access |
| Week 12 | SQS/SNS/Budget/cleanup permissions if implemented |

## Success Criteria

Week 1 is complete when:

- Root MFA is enabled.
- Root access keys are absent or reviewed and removed/inactivated.
- Root is no longer used for daily work.
- Billing credits and primary Region are reviewed.
- `thien-admin` exists and uses MFA.
- IAM groups are created by project responsibility.
- Password/MFA baseline policy is prepared.
- Team IAM users exist or are planned with individual access.
- At least one allowed operation and one denied operation are validated.
- No credentials, passwords, MFA QR codes, or access keys are stored in the repo.

## Final Hugo Layout Recommendation

Use this structure in `content/1-Worklog/1.1-Week1/_index.md`:

```text
Objective
FCAJ account context
IAM landing zone design
Implementation summary
Validation and evidence
Challenges and lessons learned
Weekly outcome
Next week plan
```

Keep the final page concise. Put implementation details in this doc, and put the strongest evidence in the Hugo page.

## Next Week Plan

Week 2 will use the secured IAM baseline to build the VPC network foundation:

- VPC CIDR design.
- Public and private subnets.
- Route tables.
- Internet Gateway.
- Security Groups.
- Network segmentation pattern for later backend, AI engine, and database workloads.

