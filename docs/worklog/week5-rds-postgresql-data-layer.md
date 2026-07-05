# Week 5 Draft - RDS PostgreSQL Data Layer

This file is the internal source and implementation guide for the Week 5 Hugo worklog page.

Target Hugo page:

```text
content/1-Worklog/1.5-Week5/_index.md
```

Public evidence folder:

```text
static/images/worklog/week5/
```

Original screenshots copied from the local FCAJ notes are stored privately under:

```text
private-evidence/week5/originals/
```

Do not embed raw screenshots directly from the Windows/WSL vault path.

## Positioning

Week 5 is now:

```text
Week 5 - RDS PostgreSQL Data Layer
```

This replaces the earlier Week 5 S3/CloudFront idea because the available evidence for this week is an RDS lab. S3/CloudFront should move to Week 6.

Week 5 focuses on:

- RDS PostgreSQL
- DB Subnet Group
- Private subnet placement
- EC2 client access
- Security Group source reference
- PostgreSQL SSL connection
- SOC-style `alerts` table and queries

## Source Material

Main source folder:

```text
/mnt/d/StorgeVault/Thiện học AI/FCAJ-Internship/01_AWS-Labs/Lab005_RDS/
```

Important source note:

```text
Primary saved evidence was consolidated around 28/05/2026.
The AWS resources were deleted later, so the page relies on saved evidence.
```

## Daily Time Model

The original one-line estimate of `5 hours` was too low for the amount of work performed. The public draft now spreads the RDS lab across multiple work sessions:

| Date | Time estimate | Focus |
| --- | --- | --- |
| 28/05/2026 | Estimated 3 hours | RDS architecture and network planning |
| 29/05/2026 | Estimated 4 hours | VPC, subnet, and Security Group preparation |
| 30/05/2026 | Estimated 4 hours | DB subnet group and RDS PostgreSQL deployment |
| 31/05/2026 | Estimated 4.5 hours | EC2 client setup and connection troubleshooting |
| 01/06/2026 | Estimated 4 hours | SOC alert schema, queries, evidence, and documentation |

Total draft estimate:

```text
19.5 hours
```

These values should remain draft estimates until the project owner confirms or adjusts them.

## Public Evidence Map

| Public image | Source image | Use |
| --- | --- | --- |
| `w5-rds-architecture.svg` | Created for report | Explains the RDS private data-layer architecture |
| `w5-e01-db-subnet-group-private-subnets.png` | `Pasted image 20260528202057.png` | DB subnet group uses private subnets in two AZs |
| `w5-e02-rds-security-group-rule.png` | `Pasted image 20260528201830.png` | RDS-SG allows PostgreSQL `5432` from EC2-SG |
| `w5-e03-rds-instance-available.png` | `Pasted image 20260528231209.png` | RDS PostgreSQL instance available, public internet gateway disabled |
| `w5-e04-postgresql-client-install.png` | `Screenshot 2026-05-28 215502.png` | PostgreSQL client installed on EC2 |
| `w5-e05-rds-ssl-connection-success.png` | `Pasted image 20260528222734.png` | `psql` connected to RDS with SSL and returned database/user |
| `w5-e06-alerts-table-created.png` | `Pasted image 20260528224155.png` | SOC-style `alerts` table was created |
| `w5-e07-alert-query-results.png` | `Pasted image 20260528224526.png` | Alert records were inserted and queried |

## Redaction Rules

Hide:

- AWS account ID
- Account alias/email if visible
- Secrets, passwords, access keys, and private key material

Keep visible:

- VPC ID, subnet ID, Security Group ID if useful
- CIDR blocks
- Availability Zones
- Database identifier
- RDS endpoint when it is needed to prove the connection path and the resource has already been cleaned up
- PostgreSQL engine/class
- `psql` output proving connection
- SQL schema/query results
- RFC 5737 documentation IPs used as sample alert values

## Evidence Interpretation

Current evidence supports:

- A DB Subnet Group was configured with private subnets in two Availability Zones.
- RDS-SG was intended to allow PostgreSQL access from EC2-SG.
- RDS PostgreSQL was available with public internet access disabled.
- PostgreSQL client was installed on EC2.
- EC2 connected to RDS using SSL.
- `alerts` table was created.
- Sample SOC alert records were inserted and queried.

Current evidence does not prove a completed backend API integration yet:

```text
Backend API endpoint returned records from RDS.
```

Keep FastAPI integration as deferred unless a public screenshot is added later.

## Suggested Missing Evidence

If the user can provide more evidence later, the strongest additions would be:

- RDS final configuration page showing `Publicly accessible: No`.
- RDS Security Group final details page after creation, not only the create form.
- EC2 instance detail showing it is in the same RDS lab VPC.
- FastAPI endpoint response reading alert records from RDS.
- Cleanup evidence showing RDS deleted and no final snapshot retained, if applicable.

## Publication Checklist

Before switching Week 5 to `draft: false`, confirm:

- Daily time spent values are real or approved as estimates.
- Public screenshots are reviewed in `static/images/worklog/week5/`.
- No raw WSL/Windows path is embedded in the Hugo page.
- No account ID is visible in public screenshots.
- No password, key, or secret is visible.
- FastAPI/RDS integration is not claimed unless evidence is added.
- Week 6 plan points to S3/CloudFront.
