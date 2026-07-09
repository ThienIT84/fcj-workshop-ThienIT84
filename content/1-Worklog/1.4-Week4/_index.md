---
title: "Week 4 - Secure Operations & Observability"
date: 2026-05-11
weight: 4
chapter: false
draft: false
pre: " <b> 1.4. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary services | AWS Systems Manager and Amazon CloudWatch |
| Supporting services | EC2 Instance Connect Endpoint, VPC Flow Logs, IAM, CloudWatch Logs |
| Main objective | Improve private infrastructure access, telemetry, and operational monitoring |
| Primary Region | `ap-southeast-1` - Asia Pacific (Singapore) |
| Infrastructure source | Existing Week 2 VPC, Week 3 EC2 pattern, and active SOC MVP migration environment |
| Session Manager | Implemented and validated |
| EIC private-IP access | Implemented and validated |
| VPC Flow Logs | Delivered records to CloudWatch Logs |
| Metric filter | Implemented for rejected network traffic |
| CloudWatch alarm | Implemented and triggered; final state captured as `ALARM` |
| Reachability Analyzer | Deferred |
| Week status | Implementation completed with selected public evidence |

## Evidence Re-validation Note

The original Week 4 operations lab notes were created on `26/05/2026`.

Additional evidence collection and missing configuration validation were completed on `30/06/2026` using the active SOC MVP migration environment. This was necessary because some original lab resources had already been deleted to control cost, and several original screenshots were not strong enough for the final FCAJ worklog.

The newer screenshots should be read as evidence re-validation, not as a claim that every Week 4 screenshot was captured on the original lab date. The page remains in draft mode until the time-spent estimates are confirmed.

## Objective

The objective of Week 4 was to improve the operational access and observability of the AWS infrastructure prepared during the previous networking and compute labs.

Instead of deploying another workload, this week focused on managing an existing EC2 instance through AWS-managed access methods, collecting VPC network telemetry, and converting rejected network traffic into a CloudWatch monitoring signal.

The implementation included AWS Systems Manager Session Manager, EC2 Instance Connect Endpoint, VPC Flow Logs, CloudWatch Logs, a custom metric filter, and a CloudWatch alarm.

## Project Context

The Hybrid IDS/AI SOC project requires more than a running backend instance. The infrastructure must also support controlled administrative access, network troubleshooting, security telemetry, and operational monitoring.

Direct SSH through a public IPv4 address was useful during the initial deployment, but it should not be the only administrative access path.

Week 4 therefore evaluated two AWS-managed access patterns:

- Session Manager for browser-based management through Systems Manager.
- EC2 Instance Connect Endpoint for temporary SSH access through a private IPv4 path.

The VPC was also configured to publish Flow Logs to CloudWatch Logs. Rejected network records were converted into a custom metric and evaluated by a CloudWatch alarm.

## Operations Architecture Overview

The Week 4 work can be read as two connected operation flows: controlled administrative access and network observability.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-operations-architecture.svg" alt="Week 4 operations architecture overview">
  <figcaption>Figure 1 - Week 4 operational access and network-observability flow for the existing SOC MVP EC2 environment.</figcaption>
</figure>

The access path validates how an administrator can reach the backend instance through AWS-managed mechanisms. The observability path validates how VPC traffic metadata can become a CloudWatch metric and then an alarm.

## Daily Worklog

| Activity | Date | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- |
| Original operations lab and notes | 11/05/2026 | Reviewed EIC Endpoint, Session Manager, VPC Flow Logs, Reachability Analyzer, and CloudWatch concepts | Operations and observability scope was defined for Week 4 | Some original resources and screenshots were not sufficient for final evidence | Re-validate missing evidence if resources are available |
| Evidence re-validation and missing implementation | 12/05/2026 | Attached an SSM IAM role, validated Session Manager, validated EIC private-IP access, configured Flow Logs delivery, set log retention, created a REJECT metric filter, created a CloudWatch alarm, and collected public-safe evidence | Secure access and network-observability controls were validated for the active SOC MVP environment | Reachability Analyzer evidence was not captured and remains deferred | Finalize screenshots, review cleanup, and continue to S3/CloudFront delivery |

## Technical Implementation Summary

Week 4 extended the existing SOC MVP infrastructure with two AWS-managed administrative access methods. The backend EC2 instance was registered with Systems Manager and accessed through Session Manager, while EC2 Instance Connect Endpoint provided an SSH path through the instance's private IPv4 address.

Network telemetry was enabled through VPC Flow Logs and CloudWatch Logs. REJECT records were transformed into the `SOC/Network` `RejectedConnections` custom metric, which triggered the `SOC-Rejected-Connections` CloudWatch alarm.

## Secure Operations Access

Session Manager and EC2 Instance Connect Endpoint solve related but different access problems.

| Access method | Purpose | Main dependency |
| --- | --- | --- |
| Session Manager | Managed browser-based administration through Systems Manager | EC2 IAM role, SSM Agent, Systems Manager connectivity |
| EC2 Instance Connect Endpoint | Temporary SSH path through a private VPC route | EIC Endpoint, Security Group path, SSH daemon, IAM permissions |

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e03-session-manager-managed-node.png" alt="Systems Manager managed node">
  <figcaption>Figure 2 - The backend EC2 instance registered as an online Systems Manager managed node.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The node appeared online with Linux and EC2 metadata.  
**Scope:** This validates managed-node registration, not application deployment.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e04-session-manager-start-session.png" alt="Session Manager shell">
  <figcaption>Figure 3 - A browser-based Session Manager shell provided administrative access without a direct local SSH session.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The shell opened as `ssm-user` and returned hostname and working-directory output.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e02-eic-private-ip-connect-success.png" alt="EIC private IPv4 connection success">
  <figcaption>Figure 4 - The existing backend EC2 instance was accessed through its private IPv4 address using EC2 Instance Connect Endpoint.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The Ubuntu shell opened through the private IPv4 target and returned user, hostname, and interface output.  
**Scope:** This validates access through the instance's private IPv4 address. It does not imply that the existing EC2 instance was moved to a private subnet.

## VPC Flow Logs and CloudWatch Logs

VPC Flow Logs were used as cloud network telemetry. They do not capture packet payloads like Wireshark or Zeek, but they provide useful metadata such as source, destination, protocol, ports, packet count, byte count, action, and log status.

For this lab, Flow Logs were delivered to CloudWatch Logs. The log group retention was limited to seven days to avoid storing lab network logs indefinitely.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e07-flow-log-record.png" alt="CloudWatch Flow Log records">
  <figcaption>Figure 5 - CloudWatch Logs received VPC Flow Log records, including accepted and rejected traffic.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** Delivered records included `ACCEPT`, `REJECT`, and `OK` status fields.  
**Redaction:** Account ID and unnecessary public IP columns were hidden while preserving action and status fields.

## Metric Filter and CloudWatch Alarm

The monitoring chain built in Week 4 was:

```text
VPC traffic
-> Flow Log record
-> CloudWatch Logs
-> metric filter
-> custom metric
-> CloudWatch alarm
```

The metric filter matched Flow Log records where `action="REJECT"` and emitted a value into the `SOC/Network` namespace.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e09-flowlogs-rejected-metric-filter.png" alt="RejectedConnections metric filter">
  <figcaption>Figure 6 - The RejectedConnections metric filter converts VPC Flow Log REJECT records into a custom CloudWatch metric.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The filter emits value `1` to the `SOC/Network` `RejectedConnections` metric when new `REJECT` log events arrive.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e08-cloudwatch-rejected-connections-alarm.png" alt="CloudWatch rejected connections alarm">
  <figcaption>Figure 7 - The SOC-Rejected-Connections alarm entered the ALARM state after the custom metric reached the configured threshold.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The alarm evaluated `RejectedConnections >= 1` for one datapoint within one minute.  
**Scope:** No SNS notification was added because this lab focused on the monitoring pipeline, not email alerting.

## Validation Matrix

| Test | Expected result | Actual result | Status |
| --- | --- | --- | --- |
| EC2 registration with Systems Manager | Instance appears online | Managed node appeared online | Pass |
| Session Manager access | Browser shell opens | Session started as `ssm-user` | Pass |
| EIC private-IP access | SSH shell opens through private IPv4 path | Ubuntu shell opened and returned host/network output | Pass |
| Flow Log delivery | Records arrive in CloudWatch Logs | `ACCEPT` and `REJECT` records were received | Pass |
| REJECT metric transformation | New `REJECT` events emit a metric value | `RejectedConnections` metric filter was created | Pass |
| Alarm evaluation | Metric greater than or equal to `1` changes alarm state | Alarm entered `ALARM` | Pass |
| Reachability Analyzer | Path analysis result is captured | Not performed with public evidence | Deferred |

Two screenshots were kept out of the main page because they were weaker evidence: the EIC endpoint detail screenshot still showed a pending state, and the Flow Log setup screenshot showed a service-role validation error. The final page uses stronger follow-up evidence instead.

## Security, Cost and Cleanup Decisions

| Decision | Reason |
| --- | --- |
| Use an EC2 IAM role | Avoid long-term AWS credentials on the instance |
| Validate Session Manager | Reduce reliance on public SSH and local `.pem` files |
| Use a Security Group reference for EIC | Restrict TCP 22 to the endpoint access path |
| Keep administrator `/32` temporarily | Avoid lockout while troubleshooting EIC access |
| Set log retention to seven days | Limit unnecessary CloudWatch Logs storage |
| Use no SNS action | Week 4 validates the monitoring pipeline only |
| Defer Reachability Analyzer | No reliable public evidence was captured |
| Review temporary resources after evidence collection | Avoid retaining unused endpoints, filters, alarms, and logs |

| Resource or control | Cleanup status |
| --- | --- |
| EIC Endpoint | Retained for evidence re-validation and follow-up review |
| VPC Flow Log | Retained while Week 4 observability evidence is reviewed |
| Metric filter | Retained with the Flow Logs log group during evidence review |
| CloudWatch alarm | Retained for Week 4 validation; no SNS action configured |
| Log group | Retained with seven-day retention |
| SSM role | Retained for backend operations |

## Issues and Troubleshooting

| Issue | Resolution |
| --- | --- |
| EIC browser connection initially failed | Verified the endpoint state and added a separate backend inbound SSH rule sourced from the EIC Endpoint Security Group |
| Existing SSH rule could not be converted from CIDR to a Security Group reference | Kept the administrator `/32` rule temporarily and added a new Security Group source rule |
| Flow Logs required a CloudWatch publishing role | Created or selected a dedicated VPC Flow Logs service role |
| The CloudWatch log group already existed | Reused the existing group and configured seven-day retention |
| The custom metric graph initially appeared empty | Opened the metric namespace, selected `RejectedConnections`, and generated new REJECT traffic |
| The new alarm initially lacked enough datapoints | Reviewed missing-data handling and generated a new post-filter REJECT event |
| Reachability Analyzer evidence was unavailable | Marked the capability as deferred instead of claiming successful validation |

Future validation should analyze the TCP/22 path between the EIC Endpoint ENI and the backend ENI.

## Lessons Learned

Secure infrastructure operations require separate controls for identity, network access, telemetry, and alerting.

Session Manager and EC2 Instance Connect Endpoint solve related but different access problems. Session Manager provides managed administrative access through Systems Manager, while EIC Endpoint provides a temporary SSH path through the VPC using a private IPv4 address.

Private IPv4 access is also not the same claim as private-subnet placement. The Week 4 EIC test proves the access path through a private IPv4 address, while subnet placement remains part of the broader VPC and compute design.

Security Group rules need to be modeled correctly. An existing IPv4 CIDR rule cannot simply be converted into a Security Group reference. A new rule must be created for the endpoint-to-instance path.

VPC Flow Logs are useful only after delivery is verified. Creating a Flow Log resource or log group is not the same as confirming that real records are being received.

The monitoring chain also contains several distinct stages. Each stage must be validated independently to avoid mistaking a configured resource for a working monitoring signal.

## Weekly Outcome

By the end of Week 4, the existing SOC MVP EC2 environment had two AWS-managed operational access methods and a basic network-security monitoring pipeline.

The EC2 instance was registered with Systems Manager and accessed through Session Manager. EC2 Instance Connect Endpoint was also used to connect to the instance through its private IPv4 address.

VPC Flow Logs delivered network records to CloudWatch Logs. Rejected traffic was transformed into a custom `RejectedConnections` metric, and a CloudWatch alarm was created and triggered to evaluate that signal.

Reachability Analyzer remained optional and was not presented as a completed validation because the corresponding evidence was not captured.

## Next Week Plan

Week 5 will move from compute operations to the managed database layer. The focus will be:

- Amazon RDS PostgreSQL
- DB Subnet Group design
- Private database placement
- EC2 client access
- Security Group source references
- PostgreSQL SSL connection
- SOC-style alert records and query validation
