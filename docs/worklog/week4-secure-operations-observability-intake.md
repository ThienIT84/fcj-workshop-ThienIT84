# Week 4 Evidence Intake - Secure Operations & Observability

Document path:

```text
docs/worklog/week4-secure-operations-observability-intake.md
```

This document is an internal checklist for collecting Week 4 evidence before implementing the public Hugo worklog page.

Target Hugo page later:

```text
content/1-Worklog/1.4-Week4/_index.md
```

Future public evidence folder:

```text
static/images/worklog/week4/
```

Do not use this file as the public report. It is only the intake and evidence planning checklist.

## Week 4 Positioning

Use the title:

```text
Week 4 - Secure Operations & Observability
```

Week 4 should focus on operating, troubleshooting, and observing the infrastructure created in Week 2 and Week 3.

Primary focus:

- EC2 Instance Connect Endpoint
- AWS Systems Manager Session Manager
- VPC Flow Logs
- Reachability Analyzer
- CloudWatch Logs, metrics, alarms, and metric filters

Do not move core EC2, NAT Gateway, or public/private compute implementation back into Week 4. Those belong to Week 3.

## Current Evidence Already Available

| Existing evidence | Status | How to use later |
| --- | --- | --- |
| `w2-e09-eic-endpoint-private-access.png` | Available | Can support EIC Endpoint / Security Group discussion |
| `w2-e11-flow-logs-cloudwatch.png` | Available | Can support CloudWatch Logs / metric filter discussion |
| `AWS Systems Manager Session Manager.md` | Notes available | Use for troubleshooting narrative unless screenshots are added |
| `Lab 4.5 EC2 Instance Connect Endpoint (EIC Endpoint).md` | Notes available | Use for EIC Endpoint explanation and troubleshooting |
| `Enable VPC Flow Logs.md` | Notes available | Use for VPC Flow Logs concept and configuration summary |
| `Using Reachability Analyzer.md` | Notes available | Use for network troubleshooting explanation unless screenshots are added |
| `CloudWatch Monitoring & Alerting.md` | Notes available | Use for CloudWatch metrics, alarms, SNS, and metric filter explanation |

Current visual evidence is enough for a draft, but not yet strong enough for a polished public Week 4 page.

## Required Evidence For A Strong Week 4 Page

| ID | Public filename | Required? | Screenshot to capture | Claim it proves | Hide | Keep visible |
| --- | --- | --- | --- | --- | --- | --- |
| W4-E01 | `w4-e01-eic-endpoint-available.png` | Strongly recommended | EC2 Instance Connect Endpoint detail page showing endpoint state `Available` or `create-complete` | EIC Endpoint was created for private EC2 access | Account ID, Owner ID, email/account alias | Endpoint ID, VPC ID, subnet ID, SG ID, state, Region |
| W4-E02 | `w4-e02-eic-private-connect-success.png` | Strongly recommended | Browser terminal or EC2 Connect screen after connecting to private EC2 through EIC Endpoint | Private EC2 can be accessed without public IPv4 or bastion | Account/user identity if visible | Private IP, hostname, `whoami`, successful shell prompt |
| W4-E03 | `w4-e03-session-manager-managed-node.png` | Recommended | Systems Manager Managed Nodes or Session Manager target list showing the EC2 instance registered | EC2 can be managed by Systems Manager prerequisites | Account ID, instance owner/account metadata | Instance ID, node status, IAM role reference, Region |
| W4-E04 | `w4-e04-session-manager-start-session.png` | Optional but strong | Session Manager shell connected to the private EC2 instance | Session Manager access worked without SSH port 22 | Account/user identity if visible | Shell prompt, hostname, managed session context |
| W4-E05 | `w4-e05-vpc-flow-logs-enabled.png` | Strongly recommended | VPC Flow Logs tab or flow log detail page showing destination `/aws/vpc/flowlogs` | VPC network telemetry was enabled | Account ID, role ARN account segment | VPC ID, log group name, destination, traffic type, status |
| W4-E06 | `w4-e06-reachability-analyzer-result.png` | Recommended | Reachability Analyzer path result showing reachable/succeeded path or failure explanation | Network path was tested using AWS analysis tooling | Account ID, owner metadata | Source, destination, protocol/port, path status, explained component |
| W4-E07 | `w4-e07-cloudwatch-nat-metric.png` | Recommended | CloudWatch NAT Gateway metric such as `PacketDropCount` or `ActiveConnectionCount` | NAT Gateway health can be monitored with CloudWatch metrics | Account ID if visible | Metric name, NAT Gateway dimension, Region, graph/result |
| W4-E08 | `w4-e08-cloudwatch-alarm.png` | Optional but strong | CloudWatch alarm detail such as `NAT-Gateway-Packet-Drops` | Alerting was prepared for NAT/network issues | Account ID, SNS email if visible | Alarm name, metric, threshold, state |
| W4-E09 | `w4-e09-flowlogs-rejected-metric-filter.png` | Strongly recommended | CloudWatch Log Group metric filter detail for `Rejected-Connections` / `RejectedConnections` | Flow Logs can be converted into a security metric | Account ID inside ARN | Log group, filter name, pattern, metric namespace/name |

Minimum public Week 4 evidence set:

```text
W4-E01 EIC Endpoint available
W4-E02 EIC private connect success
W4-E05 VPC Flow Logs enabled
W4-E06 Reachability Analyzer result
W4-E09 Flow Logs rejected metric filter
```

High-score evidence set:

```text
W4-E01 through W4-E09
```

## How To Capture Each Evidence

### W4-E01 - EIC Endpoint Available

Open:

```text
EC2 -> Network & Security -> EC2 Instance Connect Endpoints -> select endpoint
```

Capture the page where the reviewer can see:

- Endpoint name or endpoint ID
- State
- VPC
- Subnet
- Security Group
- Region

Do not crop so tightly that the page title or state disappears.

### W4-E02 - EIC Private Connect Success

Open the private EC2 instance:

```text
EC2 -> Instances -> select private EC2 -> Connect -> EC2 Instance Connect
```

Use:

```text
Connect using a Private IP
EC2 Instance Connect Endpoint
Username: ec2-user
```

After connection, capture a terminal that shows:

```bash
whoami
hostname
ip addr
```

This screenshot proves that the private instance can be reached without assigning a public IPv4 address.

### W4-E03 - Session Manager Managed Node

Open:

```text
Systems Manager -> Fleet Manager
```

or:

```text
Systems Manager -> Session Manager -> Start session
```

Capture the target instance if it appears as a managed node.

This screenshot proves the prerequisites are working:

- IAM role attached to EC2
- `AmazonSSMManagedInstanceCore`
- SSM Agent running
- Network path to Systems Manager

### W4-E04 - Session Manager Start Session

If available, start a session to the private EC2 instance and capture the shell.

Suggested commands:

```bash
whoami
hostname
pwd
```

If this is not available, Week 4 can still document Session Manager troubleshooting from lab notes, but should not claim successful session evidence.

### W4-E05 - VPC Flow Logs Enabled

Open:

```text
VPC -> Your VPCs -> select lab VPC -> Flow logs
```

Capture:

- Flow log status
- Destination type
- Destination log group `/aws/vpc/flowlogs`
- Traffic type `All`
- Aggregation interval if visible

This screenshot proves that VPC traffic metadata is being collected for troubleshooting and security monitoring.

### W4-E06 - Reachability Analyzer Result

Open:

```text
VPC -> Reachability Analyzer -> Path analyses
```

Capture one useful result:

- Reachable/succeeded result, or
- Failed result with the blocked component

Useful path examples:

```text
Public EC2 ENI -> Private EC2 ENI, TCP 22
EIC Endpoint ENI -> Private EC2 ENI, TCP 22
Private EC2 ENI -> NAT Gateway, TCP 443
```

This screenshot proves that network troubleshooting was performed with AWS analysis tooling rather than guessing.

### W4-E07 - CloudWatch NAT Metric

Open:

```text
CloudWatch -> Metrics -> All metrics -> AWS/NATGateway
```

Capture one NAT Gateway metric:

- `PacketDropCount`
- `ActiveConnectionCount`
- `BytesOutToDestination`
- `ConnectionAttemptCount`

Make sure the Region is:

```text
ap-southeast-1
```

This screenshot proves that infrastructure health can be monitored after deployment.

### W4-E08 - CloudWatch Alarm

Open:

```text
CloudWatch -> Alarms -> select NAT/network alarm
```

Capture:

- Alarm name
- Metric name
- Threshold
- Period
- Alarm state

If SNS email is visible, hide the email address.

### W4-E09 - Flow Logs Rejected Metric Filter

Open:

```text
CloudWatch -> Logs -> Log groups -> /aws/vpc/flowlogs -> Metric filters
```

Capture the metric filter:

```text
Filter name: Rejected-Connections
Metric namespace: VPC/Security
Metric name: RejectedConnections
```

The expected filter pattern is:

```text
[version, account, eni, source, destination, srcport, destport, protocol, packets, bytes, windowstart, windowend, action="REJECT", flowlogstatus]
```

This screenshot proves that VPC Flow Logs were turned into a monitoring/security metric.

## Redaction Rules

Hide:

- AWS Account ID
- `Owner ID`
- Account ID inside ARN
- Email address
- User/account alias if visible
- Access keys, secret keys, passwords, MFA QR codes

Keep visible:

- VPC ID
- EC2 instance ID
- Endpoint ID
- Security Group ID
- Subnet ID
- Route table ID
- NAT Gateway ID
- Log group name
- Metric filter name
- Metric namespace/name
- Alarm name
- CIDR blocks
- Private IPs
- Command outputs used as validation

## Public Page Claim Rules

If an evidence item is missing, Week 4 can still mention the activity, but the language must match the proof level.

Use:

```text
Documented from lab notes
Troubleshot during lab
Reviewed and prepared
```

Do not use:

```text
Verified
Validated
Successfully connected
Alarm configured
```

unless the screenshot or command output proves it.

## Suggested Week 4 Final Page Evidence Mapping

| Page section | Best evidence |
| --- | --- |
| Private Operations Access | W4-E01, W4-E02 |
| Session Manager Troubleshooting | W4-E03, W4-E04 |
| VPC Flow Logs and Telemetry | W4-E05 |
| Reachability Analyzer | W4-E06 |
| CloudWatch Monitoring | W4-E07, W4-E08 |
| Flow Logs Metric Filter | W4-E09 |

## Publication Checklist Before Implementing Week 4

Before writing the public Week 4 Hugo page, confirm:

- At least 5 strong evidence screenshots are available.
- No screenshot is merged with another screenshot.
- Every screenshot can be understood without zooming heavily.
- Account IDs and emails are hidden.
- Resource IDs and technical configuration remain visible.
- Exact daily work time is available, or the page remains `draft: true`.
- Missing evidence is clearly marked as notes/troubleshooting, not verified implementation.

## Future Test Commands

After the Week 4 Hugo page is implemented, run:

```bash
hugo --buildDrafts --destination /tmp/fcaj_hugo_week4_check
rg "static/images/week_02|/images/week_02|private-evidence" content/1-Worklog/1.4-Week4/_index.md
rg "<AWS_ACCOUNT_ID>" content/1-Worklog/1.4-Week4 docs/worklog/week4-secure-operations-observability-intake.md
```

Replace `<AWS_ACCOUNT_ID>` locally with the real 12-digit account ID when scanning. Do not write the real account ID into this checklist.

## Final Implementation Status

Week 4 was implemented using a selected public evidence set. Original unredacted screenshots were moved to:

```text
private-evidence/week4/originals/
```

Sanitized public screenshots were recreated under:

```text
static/images/worklog/week4/
```

### Published Evidence

| Public image | Status | Reason |
| --- | --- | --- |
| `w4-e02-eic-private-ip-connect-success.png` | Published | Proves private IPv4 access through EC2 Instance Connect Endpoint |
| `w4-e03-session-manager-managed-node.png` | Published | Proves Systems Manager managed-node registration |
| `w4-e04-session-manager-start-session.png` | Published | Proves browser-based Session Manager shell access |
| `w4-e05a-cloudwatch-log-group-retention.png` | Published | Proves CloudWatch log group retention was limited to one week |
| `w4-e07-flow-log-record.png` | Published | Proves Flow Log records were delivered to CloudWatch Logs |
| `w4-e08-cloudwatch-rejected-connections-alarm.png` | Published | Proves the rejected-connections alarm exists and the screenshot shows `ALARM` |
| `w4-e09-flowlogs-rejected-metric-filter.png` | Published | Proves the `RejectedConnections` metric filter exists |

### Internal Only

| Original image | Status | Reason |
| --- | --- | --- |
| `w4-e01-eic-endpoint-create-complete.png` | Internal only | The available screenshot shows the endpoint while pending, so it is not used as completed-state evidence |
| `w4-e03a-ssm-ec2-role.png` | Internal only | Useful IAM-role evidence, but managed-node and session screenshots are stronger public evidence |
| `w4-e05-vpc-flow-log-active.png` | Internal only | The screenshot includes a service-role validation error, so the final page uses delivered records and log-group evidence instead |

### Deferred

| Evidence | Status | Reason |
| --- | --- | --- |
| Reachability Analyzer path result | Deferred | No public screenshot was captured |

### Final Page Notes

- The Week 4 Hugo page remains `draft: true` until actual time spent is confirmed.
- The page states both the original lab baseline date `26/05/2026` and the evidence re-validation date `30/06/2026`.
- Reachability Analyzer is explicitly marked as deferred.
- The main page now includes an operations architecture diagram, a validation matrix, and a security/cost/cleanup decision table.
- No screenshots are merged; each public screenshot has its own figure, caption, result, and key observation.
- Alarm state should be written consistently as `ALARM`.
- EIC private IPv4 access should not be described as proof that the EC2 instance was moved to a private subnet.
