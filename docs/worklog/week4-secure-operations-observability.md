# Week 4 Draft - Secure Operations & Observability

This file is the working source and implementation guide for Week 4 before publishing to Hugo.

Target Hugo page:

```text
content/1-Worklog/1.4-Week4/_index.md
```

Public evidence folder:

```text
static/images/worklog/week4/
```

Original unredacted evidence is stored privately under:

```text
private-evidence/week4/originals/
```

The Hugo page must not embed raw screenshots from `private-evidence/` or any former `static/images/week_02/` path.

## Positioning

Use the title:

```text
Week 4 - Secure Operations & Observability
```

Week 4 focuses on operations after the VPC and EC2 work from Week 2 and Week 3:

- Systems Manager managed-node registration
- Session Manager shell access
- EC2 Instance Connect Endpoint private IPv4 access
- VPC Flow Logs delivery to CloudWatch Logs
- CloudWatch log retention
- Metric filter for rejected traffic
- CloudWatch alarm for the custom rejected-connections metric

Reachability Analyzer is deferred because no public evidence screenshot was captured.

## Evidence Timeline

Original lab notes baseline:

```text
26/05/2026
```

Evidence re-validation and missing implementation:

```text
30/06/2026
```

The Week 4 page must clearly state that newer screenshots were collected from the active SOC MVP migration environment. Do not present every screenshot as if it was captured during the original lab date.

## Publication Mode

The Hugo page should stay in draft mode until the time-spent values are confirmed:

```yaml
draft: true
```

Current draft time values:

```text
26/05/2026 - Estimated 2 hours
30/06/2026 - Estimated 4.5 hours
```

Switch to `draft: false` only after:

- Daily worklog time-spent values are confirmed by the project owner.
- Public screenshots are reviewed in `static/images/worklog/week4/`.
- Cleanup statuses are checked against the real AWS environment.
- The page has no draft-only language.
- Reachability Analyzer remains clearly marked as deferred unless evidence is added later.

## Main Page Structure

The final Week 4 page should follow this evaluator-facing structure:

1. Week at a Glance
2. Evidence Re-validation Note
3. Objective
4. Project Context
5. Operations Architecture Overview
6. Daily Worklog
7. Technical Implementation Summary
8. Secure Operations Access
9. VPC Flow Logs and CloudWatch Logs
10. Metric Filter and CloudWatch Alarm
11. Validation Matrix
12. Security, Cost and Cleanup Decisions
13. Issues and Troubleshooting
14. Lessons Learned
15. Weekly Outcome
16. Next Week Plan

## Public Evidence Map

| ID | Public image | Main page use | Notes |
| --- | --- | --- | --- |
| W4-A01 | `w4-operations-architecture.svg` | Embedded | Shows controlled access and observability flow in one diagram |
| W4-E01 | `w4-e03-session-manager-managed-node.png` | Embedded | Proves the EC2 instance registered as an online Systems Manager managed node |
| W4-E02 | `w4-e04-session-manager-start-session.png` | Embedded | Proves Session Manager shell access |
| W4-E03 | `w4-e02-eic-private-ip-connect-success.png` | Embedded | Proves EIC private IPv4 access; public source IP hidden |
| W4-E04 | `w4-e07-flow-log-record.png` | Embedded | Proves Flow Log records were delivered; account/public IP columns hidden |
| W4-E05 | `w4-e09-flowlogs-rejected-metric-filter.png` | Embedded | Proves the `RejectedConnections` metric filter exists |
| W4-E06 | `w4-e08-cloudwatch-rejected-connections-alarm.png` | Embedded | Proves the alarm reached the `ALARM` state |
| W4-E07 | `w4-e05a-cloudwatch-log-group-retention.png` | Public file available | Useful retention evidence, but not embedded in the main page to reduce page density |

Do not merge multiple screenshots into one public evidence image. Each screenshot embedded in Hugo should be a separate figure with a short caption, `Result`, and `Key observation`.

## Internal Or Deferred Evidence

| Image | Status | Reason |
| --- | --- | --- |
| `w4-e01-eic-endpoint-create-complete.png` | Internal only | The available screenshot shows the endpoint while pending, so it should not be used to claim a completed endpoint state |
| `w4-e03a-ssm-ec2-role.png` | Internal only | Useful role evidence, but managed-node and session screenshots are stronger public evidence |
| `w4-e05-vpc-flow-log-active.png` | Internal only | The available screenshot includes a service-role validation error, so use delivered records and log-group evidence instead |
| `w4-e06-reachability-eice-to-backend-ssh.png` | Deferred | No public evidence captured yet |

## Redaction Rules

Hide:

- AWS account ID
- Account ID inside ARN
- Public source IPs not needed for the claim
- Email or account alias if visible
- Access keys, secret keys, passwords, MFA QR codes

Keep visible:

- Resource names
- VPC ID, endpoint ID, Security Group ID, and instance ID when useful
- Private IP addresses
- Log group name
- Metric filter name
- Metric namespace and metric name
- Alarm name, state, and condition
- Command output proving access

## Evidence Interpretation

Current evidence supports these claims:

- The EC2 instance appeared as a Systems Manager managed node.
- A Session Manager shell was opened.
- EC2 Instance Connect Endpoint private IPv4 access worked.
- Private IPv4 access was validated, but this does not prove private-subnet placement.
- Flow Log records were delivered to CloudWatch Logs.
- CloudWatch log retention was configured to one week.
- The `RejectedConnections` metric filter was created.
- The `SOC-Rejected-Connections` alarm evaluated the custom metric and reached `ALARM`.

Current evidence does not support a completed Reachability Analyzer path-analysis claim yet:

```text
A Reachability Analyzer result proved the EIC-to-backend path.
```

Keep Reachability Analyzer marked as deferred unless a path analysis screenshot is added later.

## Security, Cost and Cleanup Notes

The public Week 4 page includes a short Solution Architect-style decision table. Keep these points visible:

- EC2 IAM role instead of long-term credentials on the instance.
- Session Manager to reduce reliance on direct SSH and local PEM files.
- Security Group reference for the EIC-to-instance TCP/22 path.
- Temporary administrator `/32` retained only to avoid lockout while troubleshooting.
- Seven-day CloudWatch Logs retention for cost control.
- No SNS action because Week 4 validates the monitoring pipeline only.
- Reachability Analyzer deferred due to missing public evidence.
- Temporary resources should be reviewed after evidence collection.

Cleanup status must be checked before publishing:

```text
EIC Endpoint: Retained / Deleted
VPC Flow Log: Retained / Deleted
Metric filter: Retained / Deleted
CloudWatch alarm: Retained / Deleted
Log group: Retained with 7-day retention / Deleted
SSM role: Retained for backend operations / Deleted
```

Do not write `Deleted` in the public page unless the resource was actually removed.

## Publication Checklist

Before changing the Hugo page to `draft: false`, confirm:

- Daily time-spent values are real or explicitly approved estimates.
- No raw `static/images/week_02/` paths are linked.
- No `private-evidence/` path is linked.
- Public images use `/images/worklog/week4/`.
- Account ID and unnecessary public IPs are hidden in every public screenshot.
- Alarm state is written consistently as `ALARM`.
- Private IPv4 access is not described as proof of private-subnet placement.
- The page does not claim Reachability Analyzer validation.
- The page clearly states the original lab date and re-validation date.
- Week 5 transition points to S3 and CloudFront.
