# Superseded Draft - Week 1 Cloud Foundation and Team IAM Access Management

> This file is kept for historical context.
>
> The current Week 1 source should be:
>
> ```text
> docs/worklog/week1-aws-account-iam-landing-zone.md
> ```
>
> The master 12-week source should be:
>
> ```text
> docs/worklog/00-fcaj-12-week-master-plan.md
> ```

Do not use this file as the current Week 1 working draft. It remains only as an archive of the earlier IAM planning direction.

Former target Hugo page:

```text
content/1-Worklog/1.1-Week1/_index.md
```

Recommended image folder:

```text
static/images/worklog/week1/
```

Images stored there can be referenced in Hugo as:

```markdown
![IAM users list](/images/worklog/week1/02-iam-users-list.png)
```

## Objective

Establish a secure AWS account foundation and implement individual, role-based access for the five project members.

The main goal is to understand AWS identity and access concepts through a practical implementation instead of only studying IAM theory.

## Current Account Context

The project is currently using one standalone AWS account with active AWS Free Tier credits.

AWS Organizations will not be enabled during this phase because creating an organization may affect the remaining credits. Therefore, individual IAM users will be used temporarily for team access.

The planned structure is:

```text
AWS Account
├── Root user
│   └── Reserved for account-level and emergency tasks
│
├── thien-admin
├── fcaj-cloud
├── fcaj-backend
├── fcaj-ai
└── fcaj-frontend
```

The root user will not be used for daily project activities.

## Services and Concepts

- AWS Identity and Access Management
- AWS root user security
- IAM users
- IAM user groups
- IAM policies
- AWS managed policies
- Customer-managed policies
- Multi-Factor Authentication
- Temporary passwords
- Least-privilege access
- Role-based access control
- AWS Management Console access
- AWS Regions
- Billing and Free Tier monitoring

## Planned IAM Users

| IAM user | Assigned role |
| --- | --- |
| `thien-admin` | Account administrator and team leader |
| `fcaj-cloud` | Cloud infrastructure and networking |
| `fcaj-backend` | FastAPI backend and deployment |
| `fcaj-ai` | Dataset, AI models, and detection worker |
| `fcaj-frontend` | Frontend deployment, monitoring, and reporting |

Each team member will receive one individual IAM user.

IAM accounts will not be shared between members.

## Planned IAM Groups

The following groups will be created:

- `FCAJ-Admins`
- `FCAJ-Cloud-Builders`
- `FCAJ-Backend-Builders`
- `FCAJ-AI-Builders`
- `FCAJ-Frontend-Builders`
- `FCAJ-Base-Users`

Planned membership:

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

Permissions will be attached to groups rather than directly to individual member users.

## Planned Activities

### Day 1 - Secure the AWS Root User

- Review the security status of the AWS root user.
- Enable or verify MFA for the root user.
- Confirm that no root access key exists.
- Confirm that the root user will not be shared with team members.
- Reserve the root user for account recovery, billing, and tasks that specifically require root credentials.
- Review the AWS Billing and Credits pages.
- Confirm the remaining AWS Free Tier credits and expiration date.

### Day 2 - Create the Administrative IAM User

- Create the `thien-admin` IAM user.
- Enable AWS Management Console access.
- Use a temporary password and require a password change at first sign-in.
- Create the `FCAJ-Admins` group.
- Attach the required administrative policy to the group.
- Add `thien-admin` to the group.
- Sign out from the root user.
- Sign in using `thien-admin`.
- Change the temporary password.
- Enable MFA for `thien-admin`.
- Use `thien-admin` for subsequent account administration.

### Day 3 - Design the Team Permission Structure

- Identify the responsibility of each project member.
- Create the required IAM groups.
- Define which AWS services each group will need during the project.
- Create the `FCAJ-Base-Users` group for common user security permissions.
- Avoid granting `AdministratorAccess` or `PowerUserAccess` to all members.
- Prepare a permission matrix for the five users.
- Select `ap-southeast-1` as the primary Region for project resources.

Initial builder permissions will remain limited. Additional service permissions will be added during the corresponding project weeks.

### Day 4 - Create Team IAM Users

Create the following IAM users:

- `fcaj-cloud`
- `fcaj-backend`
- `fcaj-ai`
- `fcaj-frontend`

For each member:

- Enable AWS Management Console access.
- Generate a temporary password.
- Require password reset at first sign-in.
- Add the user to `FCAJ-Base-Users`.
- Add the user to the correct role-based group.
- Do not create an access key unless there is a verified technical requirement.
- Record the assigned user and group without recording passwords.

Each member will receive privately:

- IAM sign-in URL
- IAM username
- Temporary password

### Day 5 - Member Onboarding and Permission Validation

Each member will:

- Sign in using their individual IAM user.
- Change the temporary password.
- Register their own MFA device.
- Sign out and sign in again using MFA.
- Test an operation that should be allowed.
- Test an operation that should be denied.

Example validation:

```text
Allowed test:
Open the EC2 or VPC Console and view existing resources.

Denied test:
Attempt to create an IAM user or modify IAM policies.

Expected result:
AccessDenied
```

The tests will demonstrate that permissions are assigned according to job responsibility and that members cannot manage identities or resources outside their assigned scope.

## Permission Expansion Plan

Permissions will be expanded gradually according to the weekly project tasks.

| Week | Permission to be added |
| --- | --- |
| Week 1 | Identity, sign-in, MFA, and basic read access |
| Week 2 | VPC and network lab permissions |
| Week 3 | EC2, Session Manager, and monitoring permissions |
| Week 4 | S3 and CloudFront permissions |
| Week 5 | RDS-related permissions |
| Week 6 | CloudWatch and troubleshooting permissions |
| Week 10 | Backend deployment and artifact access |
| Week 11 | SQS and SNS permissions |
| Week 12 | Final review and permission cleanup |

This approach applies least privilege by granting access only when a member has an active project requirement.

## Expected Deliverables

By the end of Week 1, the following results should be available:

- Root user protected with MFA.
- Root user excluded from daily operations.
- One administrative IAM user created for the account owner.
- Five individual IAM users available for the five project members.
- Role-based IAM groups created.
- Users assigned to the correct groups.
- Temporary passwords changed by members.
- MFA enabled for active users.
- No shared IAM accounts.
- No unnecessary access keys.
- At least one successful allowed-operation test.
- At least one expected `AccessDenied` test.
- A documented IAM permission matrix.
- Screenshots and notes prepared for the FCAJ worklog.

## Evidence to Collect

- Root MFA status.
- Root access-key status.
- IAM users list.
- IAM groups list.
- User-to-group membership.
- Policies attached to each group.
- `thien-admin` configuration.
- MFA status for IAM users.
- Successful IAM user sign-in.
- Successful allowed-operation test.
- Expected `AccessDenied` result.
- AWS Region selection.
- Billing and credit status.
- Team permission matrix.

All evidence must hide sensitive information, including:

- Passwords
- MFA QR codes
- MFA secret keys
- Access keys
- Secret access keys
- Personal email addresses
- Unnecessary account details

## Success Criteria

Week 1 will be considered complete when:

- The root user is protected and no longer used for routine work.
- All five project members have separate IAM identities.
- Each member has changed their temporary password.
- Each active user has enabled MFA.
- Each user belongs to the correct IAM groups.
- Administrative access is limited to the designated administrator.
- Members can perform approved operations.
- Members receive `AccessDenied` for unauthorized operations.
- No credentials are shared or stored in the project source code.
- All implementation evidence has been collected and organized.

## Worklog Direction

This week focuses on establishing a secure cloud and identity foundation before deploying project infrastructure.

The practical outcome is a team access model in which every member has an individual identity, permissions are assigned through role-based groups, MFA is required, and unauthorized operations are restricted according to the principle of least privilege.

The main lesson is that AWS security begins with identity design. Compute, storage, database, and networking resources should only be deployed after user access and administrative responsibilities have been clearly defined.

## Recommended Final Hugo Layout

When this draft is converted into the final worklog page, use a concise evaluator-friendly structure:

```text
1. Objective
2. Account context and constraints
3. IAM design
4. Implementation
5. Validation
6. Evidence
7. Challenges and lessons learned
8. Outcome and next step
```

Do not put every screenshot in one long block. Place each screenshot near the claim it proves.

## Image Placement Plan

Use this section to map screenshots to the final page layout. Add a short note after each image if it contains sensitive areas that need cropping or masking.

| Image file | What it should show | Suggested section | Caption idea |
| --- | --- | --- | --- |
| `01-root-mfa-status.png` | Root user MFA enabled | Day 1 / Evidence | Root user was protected with MFA and reserved for account-level tasks. |
| `02-root-access-key-status.png` | No root access key exists | Day 1 / Evidence | Root user access keys were not created for routine project activities. |
| `03-billing-credit-status.png` | Free Tier or credit status | Current Account Context | Billing and credit status was reviewed before creating lab resources. |
| `04-iam-users-list.png` | IAM users for the team | Planned IAM Users / Evidence | Individual IAM users were created for project members instead of shared accounts. |
| `05-iam-groups-list.png` | IAM groups created for roles | Planned IAM Groups / Evidence | Role-based IAM groups were created for administration, cloud, backend, AI, and frontend tasks. |
| `06-group-membership-admin.png` | `thien-admin` group membership | Planned IAM Groups | The administrator account belongs to the base group and admin group. |
| `07-group-membership-builders.png` | Builder users in their groups | Planned IAM Groups | Each builder user was mapped to the group matching project responsibility. |
| `08-group-policy-assignment.png` | Policies attached to groups | Permission Expansion Plan | Permissions were attached at group level instead of directly to individual users. |
| `09-admin-user-config.png` | `thien-admin` console access/MFA | Day 2 / Evidence | The administrative IAM user was configured for console access and MFA. |
| `10-member-mfa-status.png` | IAM users with MFA enabled | Day 5 / Validation | Active users registered MFA devices before continuing project tasks. |
| `11-user-sign-in-success.png` | Successful IAM user console sign-in | Validation | A member successfully signed in with their individual IAM identity. |
| `12-allowed-operation-test.png` | Successful allowed read operation | Validation | The user could perform an approved read-only operation. |
| `13-access-denied-test.png` | Expected `AccessDenied` result | Validation | An unauthorized IAM/resource operation was denied as expected. |
| `14-region-selection.png` | `ap-southeast-1` selected | Current Account Context / Evidence | The Singapore Region was selected as the primary project Region. |
| `15-permission-matrix.png` | Team permission matrix | Permission Expansion Plan | Access will be expanded gradually based on weekly project responsibilities. |

## If Screenshots Have Random Names

It is acceptable if screenshots are not named clearly at first.

Preferred workflow:

```text
1. Paste/upload screenshots in chat, or place them in a temporary folder.
2. I inspect the image content.
3. I classify each screenshot by evidence type.
4. I suggest a clean filename.
5. You or I place it under static/images/worklog/week1/.
6. The final Hugo page references the renamed image.
```

Example:

```text
Original screenshot name:
Screenshot 2026-06-28 153012.png

After classification:
04-iam-users-list.png
```

If several images look similar, include one short note when sending them:

```text
These are IAM user creation screenshots.
This one is the AccessDenied test.
This one is billing/credit status.
```

Even without notes, I can usually infer the screenshot type from the AWS Console page title, sidebar, table headings, and visible success/error messages. Naming the images is still useful later because Hugo pages need stable image paths.

## Screenshot Handling Rules

- Do not include passwords, MFA QR codes, secret access keys, or full account IDs.
- Crop browser tabs and personal browser profile areas if possible.
- If an account ID appears, blur or cover most digits.
- Prefer one clear screenshot per evidence point instead of many similar screenshots.
- Use numbered filenames so the report order is obvious.
- Keep the final page readable. If there are too many screenshots, move extra screenshots to an appendix or evidence gallery.

## High-Scoring Worklog Checklist

To make this week strong for evaluation, the page must prove four things:

### 1. There Was a Real Problem

Explain why IAM mattered before creating AWS resources:

```text
The team needed individual access to one shared AWS account while protecting root credentials and Free Tier credits.
```

### 2. There Was a Clear Design

Show the access model:

```text
Root user for emergency/account tasks.
thien-admin for administration.
Role-based IAM users and groups for project members.
Permissions expanded gradually by project week.
```

### 3. There Was Real Validation

Do not only say that users and groups were created. Show both positive and negative tests:

```text
Allowed: member can view approved resources.
Denied: member cannot create IAM users or modify policies.
```

### 4. There Was a Lesson Learned

End with a short reflection:

```text
IAM is not only an account setup step. It controls how safely the team can build every later service, including VPC, EC2, S3, RDS, and SQS.
```

## Final Page Notes

The final page should tell this story:

```text
Problem:
The team needed safe AWS access under one account without sharing root credentials.

Action:
Root was secured, IAM users were created, groups were designed by responsibility, MFA was required, and permissions were attached to groups.

Validation:
The team tested sign-in, MFA, allowed operations, and expected AccessDenied behavior.

Outcome:
A secure identity baseline was ready for Week 2 VPC and networking labs.
```
