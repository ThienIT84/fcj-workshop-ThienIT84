---
title: "Week 5 - RDS PostgreSQL Data Layer"
date: 2026-05-28
weight: 5
chapter: false
draft: false
pre: " <b> 1.5. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary service | Amazon RDS |
| Database engine | PostgreSQL |
| Supporting services | VPC, DB Subnet Group, EC2, Security Groups |
| Main objective | Build a private managed database layer for SOC alert data |
| Primary Region | `ap-southeast-1` - Asia Pacific (Singapore) |
| Network model | Public EC2 client to private RDS PostgreSQL |
| Database access model | RDS Security Group allows PostgreSQL `5432` from the EC2 Security Group |
| Project output | `alerts` table with severity, message, and anomaly-score fields |
| Week status | Implemented with selected public evidence; time estimate requires owner confirmation |

## Evidence Source Note

The Week 5 RDS lab evidence was consolidated on `28/05/2026` in:

```text
FCAJ-Internship/01_AWS-Labs/Lab005_RDS/
```

The implementation effort was larger than a single short session. The saved screenshots represent the final captured evidence, while the daily worklog below spreads the actual RDS lab effort across multiple work sessions.

The original AWS resources had already been deleted before this worklog page was prepared, so the page reuses the saved lab notes and screenshots as evidence. Raw screenshots are kept privately, while the public page uses sanitized images under:

```text
static/images/worklog/week5/
```

## Objective

The objective of Week 5 was to design and validate a managed relational data layer for the Hybrid IDS/AI SOC project.

Raw logs, datasets, and model artifacts are better suited for object storage, but final alerts, anomaly scores, and dashboard records need structured queries. Amazon RDS PostgreSQL was selected as the managed database service for this structured data layer.

## Project Context

The SOC project needs a place to persist security findings after raw telemetry has been processed by the backend or AI worker.

In the project design, Zeek, Suricata, or VPC Flow Log events can be transformed into alert records. Those records can then be stored in PostgreSQL and queried by a backend dashboard.

Week 5 focused on this access pattern:

```text
Laptop
-> SSH to public EC2 client
-> psql over PostgreSQL 5432 with SSL
-> private RDS PostgreSQL
-> alerts table for SOC-style records
```

The public EC2 instance was used as a temporary database client for the learning lab. In the target SOC architecture, the application backend should use the same Security Group–based database access pattern from a private subnet, without requiring the database itself to be publicly accessible.

| Deployment | Access path |
| --- | --- |
| Learning lab | Laptop → public EC2 client → private RDS |
| Target architecture | Application backend (private subnet) → private RDS |

Database credentials were used only for the lab connection and were not embedded in the Hugo report, screenshots, source code, or Git repository.

## RDS Architecture Overview

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-rds-architecture.svg" alt="RDS PostgreSQL architecture overview">
  <figcaption>Figure 1 - Week 5 RDS PostgreSQL architecture for storing SOC alerts and anomaly scores.</figcaption>
</figure>

The database is placed behind the VPC boundary and accessed through a controlled Security Group path. The EC2 client is used as the test application host, while RDS remains private and is reached through the database endpoint.

## Daily Worklog

| Activity | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| RDS architecture and network planning | 28/05/2026 | Estimated 3 hours | Reviewed RDS PostgreSQL use cases, planned a dedicated VPC, public/private subnet layout, DB subnet group, and EC2 client access path | RDS lab architecture was defined around private database placement | RDS should not be public; EC2 client can stay public for controlled lab access | Build network prerequisites |
| VPC, subnet, and Security Group preparation | 29/05/2026 | Estimated 4 hours | Created the RDS lab VPC, prepared public/private subnets, created EC2 and RDS Security Groups, and configured PostgreSQL `5432` access from EC2-SG to RDS-SG | Network and access-control baseline was ready for RDS | Security Group references only work correctly within the same VPC | Create DB subnet group and RDS instance |
| DB subnet group and RDS PostgreSQL deployment | 30/05/2026 | Estimated 4 hours | Created the DB subnet group with private subnets across two Availability Zones and deployed a cost-aware RDS PostgreSQL instance | RDS PostgreSQL became available with public access disabled | Disabled production-cost features that were not required for the lab | Prepare EC2 client and test database access |
| EC2 client setup and connection troubleshooting | 31/05/2026 | Estimated 4.5 hours | SSHed into EC2, installed PostgreSQL client tools, tested `psql`, fixed timeout/authentication issues, and added `sslmode=require` | EC2 connected to RDS PostgreSQL over SSL | Timeout pointed to network/Security Group issues; no-encryption errors required SSL mode | Create SOC alert schema and run queries |
| SOC alert schema, query validation, and documentation | 01/06/2026 | Estimated 4 hours | Created the `alerts` table, inserted sample SOC events, queried records by severity and anomaly score, captured final evidence, and documented lessons learned | RDS stored and returned SOC-style alert data successfully | FastAPI integration needs separate evidence before it can be claimed | Use S3/CloudFront next for static delivery and artifact storage |

## Technical Implementation Summary

Week 5 created a dedicated RDS lab network using CIDR `10.20.0.0/16` with public and private subnets across two Availability Zones.

A DB Subnet Group was created from private subnets in `ap-southeast-1a` and `ap-southeast-1b`, providing RDS with eligible private subnet placement across two Availability Zones. A public EC2 instance acted as the client host for SSH access and database testing.

RDS PostgreSQL was deployed as a small lab instance with public access disabled. The database Security Group allowed PostgreSQL traffic on port `5432` only from the EC2 client Security Group.

After installing the PostgreSQL client on EC2, the connection to RDS was tested using `sslmode=require`. The lab then created an `alerts` table and inserted sample SOC records such as SSH brute force and port scan events.

## Network and Database Placement

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e01-db-subnet-group-private-subnets.png" alt="DB subnet group private subnets">
  <figcaption>Figure 2 - The DB subnet group selected private subnets across two Availability Zones.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The subnet group uses private subnets in `ap-southeast-1a` and `ap-southeast-1b` with CIDRs `10.20.128.0/20` and `10.20.144.0/20`.  
**Security value:** RDS placement is controlled separately from access permissions.

> [!NOTE]
> The DB Subnet Group covered private subnets in two Availability Zones, but the lab database itself used a single-instance deployment. This subnet coverage should not be interpreted as a validated Multi-AZ database deployment.

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e02-rds-security-group-rule.png" alt="RDS Security Group PostgreSQL rule">
  <figcaption>Figure 3 - The RDS Security Group allowed PostgreSQL 5432 from the EC2 client Security Group.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The inbound PostgreSQL rule uses an EC2 Security Group reference instead of `0.0.0.0/0`.  
**Scope:** This proves the intended database access path, not a production-grade network policy.

## RDS PostgreSQL Instance

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e03-rds-instance-available.png" alt="RDS PostgreSQL available">
  <figcaption>Figure 4 - The RDS PostgreSQL instance was available with public internet access disabled.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The database engine was PostgreSQL, the instance class was `db.t3.micro`, and public accessibility was disabled (`Publicly accessible: No`).  
**Cost note:** The lab used a small instance class and avoided unnecessary high-availability or advanced monitoring features.

## EC2 Client and Database Connection

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e04-postgresql-client-install.png" alt="PostgreSQL client installed on EC2">
  <figcaption>Figure 5 - PostgreSQL client tools were installed on the EC2 client instance.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The EC2 host installed `postgresql15` and confirmed `psql` version `15.16`.

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e05-rds-ssl-connection-success.png" alt="RDS SSL connection success">
  <figcaption>Figure 6 - The EC2 client connected to RDS PostgreSQL using SSL and queried the current database/user.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The `psql` session reached `fcaj_db`, used SSL, and returned the current database and user.  
**Troubleshooting value:** Earlier authentication/connection attempts are visible, showing that the final successful command came after correcting the connection string and SSL mode.

## SOC Alert Data Model

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e06-alerts-table-created.png" alt="Alerts table created in PostgreSQL">
  <figcaption>Figure 7 - The `alerts` table was created with event type, severity, IP fields, message, anomaly score, and timestamp.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The table schema matches SOC dashboard needs rather than a generic database demo.

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e07-alert-query-results.png" alt="Alert query results in PostgreSQL">
  <figcaption>Figure 8 - Sample alert records were inserted and queried by severity and anomaly score.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** Query results show `SSH brute force` and `Port scan` records with severity and anomaly-score values.

## Validation Matrix

| Test | Expected result | Actual result | Status |
| --- | --- | --- | --- |
| DB subnet placement | RDS subnet group uses private subnets across two AZs | Private subnets selected in `ap-southeast-1a` and `ap-southeast-1b` | Pass |
| RDS access control | PostgreSQL `5432` is allowed from EC2-SG, not from the public internet | RDS-SG used EC2 Security Group source | Pass |
| RDS availability | PostgreSQL instance becomes available | `fcaj-postgres-db` appeared available | Pass |
| Public database access | Database is not publicly accessible | `Publicly accessible: No` | Pass |
| EC2 database client | `psql` is installed on EC2 | `psql (PostgreSQL) 15.16` confirmed | Pass |
| SSL database access | EC2 connects to RDS using SSL | `psql` connected with SSL and returned `fcaj_db` | Pass |
| SOC data persistence | Alert table stores queryable records | `alerts` table inserted and returned two sample records | Pass |
| FastAPI integration | Backend API reads records from RDS | Not supported by public Week 5 screenshot evidence | Deferred |

## Issues and Troubleshooting

| Issue | Resolution |
| --- | --- |
| Initial RDS connection timed out | Checked whether EC2 and RDS Security Groups were in the same VPC and whether RDS-SG allowed PostgreSQL `5432` from EC2-SG |
| EC2-SG and RDS-SG were not aligned correctly | Recreated or corrected Security Group usage so both resources followed the same RDS lab VPC model |
| Password authentication failed | Rechecked database username/password handling and retried the `psql` connection |
| An earlier connection attempt did not request the required SSL mode | Added `sslmode=require` to the PostgreSQL connection string and confirmed an encrypted session |
| RDS can become expensive if left running | Deleted lab resources after collecting evidence and kept the page evidence-based rather than claiming active resources still exist |

## Security, Cost and Cleanup Decisions

| Decision | Reason |
| --- | --- |
| Place RDS in private subnets | Database should not be directly exposed to the internet |
| Use DB Subnet Group | Control where the managed database can be placed |
| Use EC2-SG as RDS-SG source | Restrict database access to the application/client path |
| Use `sslmode=require` for `psql` connection | Validate an encrypted PostgreSQL session during the lab |
| Use a small `db.t3.micro` lab instance | Keep the lab cost-aware |
| Use a Single-AZ database deployment and omit RDS Proxy for the lab | Useful in production, but unnecessary for short validation |
| Disable deletion protection for the lab | Allows cleanup after evidence collection |
| Store raw logs elsewhere | RDS is for queryable alert records, not large raw log files |
| Keep database credentials outside screenshots and source code | Prevent credentials from entering the public report or repository |
| Use temporary manual credentials for the lab | Secrets Manager integration was outside the Week 5 scope |
| Defer application secret management to the migration phase | The migration sprint can introduce Secrets Manager or Parameter Store |

## Data Placement Decision

Not all SOC data belongs in a relational database. The Week 5 lab used PostgreSQL for final alert records, while raw telemetry and model artifacts are better placed in object storage.

| Data type | Recommended storage | Reason |
| --- | --- | --- |
| Raw Zeek/Suricata logs | Amazon S3 | Large, append-oriented artifacts suited for long-term retention |
| VPC Flow Log archives | Amazon S3 | High-volume network records better stored as objects |
| Model artifacts | Amazon S3 | Object-based deployment artifacts |
| Final alerts | PostgreSQL | Structured queries, filtering, sorting, and dashboard persistence |
| Alert scores and status | PostgreSQL | Backend and dashboard need relational queries |
| Temporary processing data | Local/ephemeral storage | Intermediate data does not always need durable relational storage |

## Lessons Learned

DB Subnet Groups and Security Groups solve different problems. The DB Subnet Group decides where RDS can be placed, while the RDS Security Group decides who can connect to it.

Connection timeouts usually point to networking or Security Group issues, not passwords. Authentication errors and SSL errors appear only after the network path reaches the database.

RDS endpoints should be used by applications instead of fixed database IP addresses. The endpoint remains the stable application-facing address even if AWS changes the underlying database host.

For this SOC project, PostgreSQL is a good fit for final alert records because the dashboard needs structured filtering, sorting, aggregation, and severity-based queries.

`sslmode=require` encrypted the database session. A production hardening step would additionally validate the RDS certificate chain and hostname with `sslmode=verify-full`, rather than treating transport encryption alone as complete server-identity verification.

## Weekly Outcome

By the end of Week 5, a private RDS PostgreSQL data-layer pattern was implemented and validated with EC2-based client access.

The lab proved that RDS can store structured SOC alert records while staying behind a private database access path. The EC2 client connected using PostgreSQL over SSL, created an `alerts` table, inserted sample findings, and queried the records by severity and anomaly score.

FastAPI/RDS integration remains a follow-up item unless public evidence is added later.

## Next Week Plan

Week 6 will move from structured database storage to object storage and static delivery. The focus will be:

- Amazon S3 bucket design
- Private raw object storage for SOC-style datasets
- S3 static website hosting for selected website files
- Selective bucket policy instead of broad bucket-wide public access
- Amazon CloudFront distribution using the S3 website endpoint as origin
- CloudFront URL validation
- S3 Versioning and Lifecycle rules
- Clear separation between raw private objects and public website assets
