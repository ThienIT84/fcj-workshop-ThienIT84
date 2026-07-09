---
title : "Clean up resources"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Clean up resources

This section explains how to clean up the SOC AI workshop environment after the
demo. The runbook is **AWS Console first**: capture evidence with screenshots,
then remove resources in dependency order.

> **Before publishing:** screenshots in this section may include account IDs,
> usernames, ARNs, domains, public IPs, or real resource names. Mask sensitive
> information before publishing the workshop site.

#### Cleanup goals

+ Preserve required evidence before deletion.
+ Delete resources in dependency order.
+ Avoid ongoing cost from EC2, ALB, NAT Gateway, RDS, SQS, S3, CloudFront, and
  CloudWatch.
+ Record whether each resource is deleted, retained, blocked, or deferred.
+ Do not force cleanup while AWS blocks the action because of a billing or
  pricing-plan constraint.

#### Safety rules

Before deleting any resource, confirm:

```text
[ ] The AWS account is correct.
[ ] The regional services are in ap-southeast-1.
[ ] CloudFront/WAF is checked in Global/CloudFront scope.
[ ] Evidence is archived outside resources that will be deleted.
[ ] No producer, replay script, or shipper is sending new events.
[ ] The required approval gate is recorded for destructive operations.
```

Recommended approval gates:

```text
APPROVAL_DELETE_RDS_WITH_FINAL_SNAPSHOT
APPROVAL_DELETE_S3_DATA_BUCKET_AFTER_EVIDENCE_BACKUP
APPROVAL_PURGE_OR_DELETE_DLQ_AFTER_FORENSIC_EVIDENCE
APPROVAL_FULL_NETWORK_TEARDOWN
```

#### Cleanup status summary

| Area | Status | Note |
|---|---|---|
| EC2 backend/worker | `DELETED` | Terminated after services and evidence were handled. |
| ALB / Target Group | `DELETED` | Removed after compute no longer needed traffic. |
| SQS main queue / DLQ | `DELETED` | Deleted after queue/DLQ evidence was saved. |
| S3 data/frontend buckets | `DELETED` or `RETAINED` | Delete the data bucket only after archiving evidence/model artifacts. |
| RDS PostgreSQL | `DELETED WITH FINAL SNAPSHOT` | Export or screenshot `final_alerts` before deletion. |
| Secrets Manager | `SCHEDULED DELETION` | Use a recovery window; do not force delete. |
| CloudFront distribution | `PARTIAL / DEFERRED` | Blocked by CloudFront flat-rate pricing plan. |
| CloudFront WAF Web ACL | `PARTIAL / DEFERRED` | Web ACL is required while the distribution remains under the pricing plan. |
| CloudFront OAC | `BLOCKED / DEFERRED` | Do not delete OAC while the distribution still references it. |
| IAM Identity Center | `REVIEWED` | Do not treat team access as traditional IAM users. |
| NAT/EIP/VPC | `DELETED` or `RETAINED` | Full network teardown requires explicit approval. |
| KMS | `RETAINED` by default | Schedule deletion only if no encrypted evidence/snapshot is still needed. |

#### 1. Preserve evidence before cleanup

Before stopping services or deleting resources, save at least:

```text
[ ] Dashboard or /api/alerts/latest evidence.
[ ] D5-5 idempotency PASS evidence.
[ ] RDS final_alerts query/export.
[ ] SQS/DLQ queue state.
[ ] S3 evidence/model artifact inventory.
[ ] CloudWatch backend/worker logs.
[ ] CloudFront/WAF/OAC state.
[ ] IAM Identity Center users/groups/assignments.
[ ] Billing or pricing-plan state if CloudFront is locked.
```

RDS `final_alerts` evidence before deleting the database:

![RDS final_alerts evidence](/images/5-Workshop/5.6-Cleanup/socai-cleanup/10-rds-final-alerts-evidence.png)

*Caption:* The `final_alerts` table is still queryable before cleanup and shows
the SQL Injection and Cross-Site Scripting alerts used in the demo.

#### 2. Stop backend, worker, and EC2

Console path:

```text
EC2 -> Instances
```

Steps:

1. Stop producers, replay scripts, and tailers.
2. Stop the worker after the SQS main queue is drained or the queue state is
   captured.
3. Stop the backend when no new event ingestion is required.
4. Terminate backend and worker EC2 instances after logs/evidence are saved.

![Terminate backend and worker instances](/images/5-Workshop/5.6-Cleanup/socai-cleanup/01-ec2-terminate-backend-worker.png)

*Caption:* The `backend` and `ai-worker` EC2 instances are selected for
termination after evidence and queue state have been handled.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| backend EC2 terminated | DONE | E02 | Instance terminated after logs saved |
| ai-worker EC2 terminated | DONE | E02 | Instance terminated after queue state handled |

#### 3. Delete ALB and Target Group

Console path:

```text
EC2 -> Load Balancers
EC2 -> Target Groups
```

Order:

1. Confirm no new requests need to route through the ALB.
2. Delete or detach remaining listeners/rules if needed.
3. Delete the ALB.
4. Delete the target group after no load balancer references it.

![Delete ALB action](/images/5-Workshop/5.6-Cleanup/socai-cleanup/02-alb-delete-menu.png)

![Confirm ALB deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/03-alb-delete-confirm.png)

![Delete target group](/images/5-Workshop/5.6-Cleanup/socai-cleanup/04-target-group-delete-menu.png)

*Caption:* The `socai-dev-alb` ALB and `socai-dev-backend-tg` target group are
removed after compute no longer needs to receive traffic.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| ALB `socai-dev-alb` deleted | DONE | E03 | Listeners removed, ALB deleted |
| Target group `socai-dev-backend-tg` deleted | DONE | E04 | TG removed after ALB deletion |

#### 4. Delete SQS main queue and DLQ

Console path:

```text
Amazon SQS -> Queues
```

Steps:

1. Save screenshots of queue attributes and message counts.
2. Inspect the DLQ if messages remain.
3. Purge/delete the DLQ only after forensic evidence and approval.
4. Delete the main queue.
5. Delete the DLQ.

> `PurgeQueue` is irreversible. Do not purge the main queue or DLQ before
> forensic evidence and approval are recorded.

If the queue itself will be deleted, purging is usually not required. Purge is
only needed when the queue must be retained but all messages must be removed.
In either case, inspect and save DLQ evidence before any purge or delete action.

![Main SQS queue deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/05-sqs-main-queue-deleted.png)

![DLQ deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/06-sqs-dlq-deleted.png)

*Caption:* The main queue is deleted first, then the DLQ is deleted after queue
evidence and the DLQ decision are complete.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| SQS main queue deleted | DONE | E05 | Deleted after queue evidence saved |
| SQS DLQ deleted | DONE | E06 | Deleted after forensic decision and approval |

#### 5. Archive and delete S3 buckets

Console path:

```text
S3 -> Buckets
```

The data/artifact bucket must be handled carefully because it may contain
evidence, model artifacts, D5-5 receipts, report screenshots, and export files.

Checklist:

```text
[ ] Evidence/model artifacts are archived outside the bucket.
[ ] Current objects are emptied.
[ ] If versioning is enabled, object versions are deleted.
[ ] Delete markers are deleted.
[ ] The bucket is confirmed empty.
[ ] The bucket is deleted.
```

![S3 data bucket content before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/07-s3-data-bucket-contents.png)

![S3 delete objects status](/images/5-Workshop/5.6-Cleanup/socai-cleanup/08-s3-delete-objects-status.png)

![S3 buckets deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/09-s3-buckets-deleted.png)

*Caption:* The data bucket is reviewed before deletion. Objects are then deleted
successfully, and the frontend/static bucket is removed from the bucket list.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| S3 data/artifact bucket emptied and deleted | DONE | E07, E08 | Objects deleted, versions and delete markers cleared |
| S3 frontend/static bucket deleted | DONE | E08 | Bucket removed from list |

#### 6. Delete RDS with a final snapshot

Console path:

```text
RDS -> Databases
```

Required gate:

```text
[ ] APPROVAL_DELETE_RDS_WITH_FINAL_SNAPSHOT recorded.
[ ] final_alerts evidence saved.
[ ] Deletion protection checked.
[ ] Create final snapshot enabled.
[ ] Final snapshot name is timestamped or clearly named.
```

![Delete RDS with final snapshot](/images/5-Workshop/5.6-Cleanup/socai-cleanup/11-rds-delete-final-snapshot.png)

![RDS delete confirmation](/images/5-Workshop/5.6-Cleanup/socai-cleanup/12-rds-delete-confirmation.png)

![RDS deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/13-rds-deleted.png)

*Caption:* The `socai-dev-postgres-01` database is deleted with a final
snapshot. After deletion, the RDS console shows no workshop DB instance.

> **Cost note:** Deleting the DB instance stops instance runtime charges.
> However, retained automated backups and manual snapshots can still incur
> storage charges until deleted. The final snapshot is intentionally retained
> as evidence and recovery material. Review and delete after the
> workshop/report retention period ends.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| RDS `socai-dev-postgres-01` DB instance deleted | DONE | E09 | Deleted with final snapshot |
| Final snapshot created and retained | RETAINED | E09 | Evidence/recovery material — delete after retention period |

#### 7. Schedule deletion for Secrets Manager

Console path:

```text
Secrets Manager -> Secrets
```

Steps:

1. Open the secret metadata.
2. Do not reveal or screenshot the secret value.
3. Delete/schedule deletion only after backend/worker are stopped and the RDS
   snapshot is safe.
4. Use a recovery window; do not force delete.

![Secrets Manager list before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/14-secrets-manager-list.png)

![Secret metadata before delete](/images/5-Workshop/5.6-Cleanup/socai-cleanup/15-secret-before-delete.png)

![Secret scheduled for deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/16-secret-scheduled-deletion.png)

*Caption:* The `socai-dev/rds/app` secret is scheduled for deletion with a
recovery window. The secret value is not shown in evidence. Secrets scheduled
for deletion are inaccessible immediately but can be recovered before the
recovery window ends. Secrets marked for deletion do not incur additional charges.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| `socai-dev/rds/app` secret scheduled for deletion | DONE | E10 | Recovery window active — secret inaccessible, value not shown |

#### 8. CloudFront, OAC, and WAF: cleanup deferred

This section documents both the normal CloudFront cleanup flow and the actual
SOC AI cleanup result. In this workshop, final deletion is deferred because
the distribution is still locked by a CloudFront flat-rate pricing plan.

Console paths:

```text
CloudFront -> Distributions
CloudFront -> Security -> Origin access
WAF & Shield -> Protection packs (web ACLs)
```

#### CloudFront inventory found during cleanup

| Distribution ID | Domain | Purpose | Active Web ACL | Cleanup status |
|---|---|---|---|---|
| E2FO8ADXYHNOSW | dj9itpaxbbdgy.cloudfront.net | SOC AI dev CloudFront | CreatedByCloudFront-be875b92 | Deferred – flat-rate plan lock |
| E1UCULAFOQB3ZB | d2s3u1yztzei92.cloudfront.net | soc-platform-frontend | CloudFront-created Web ACL | Transitioning to pay-as-you-go |
| soc-platform-production-web-acl | N/A | Custom WAF (not active) | Not associated with tested distribution | Do not assume it protects active CloudFront |

> `soc-platform-production-web-acl` is **not** the active Web ACL for E2FO8ADXYHNOSW.
> The active Web ACL is `CreatedByCloudFront-be875b92`. Do not attempt WAF rule
> changes on `soc-platform-production-web-acl` to affect this distribution.

#### Expected normal cleanup path

```text
Disable distribution → wait Status = Deployed → disassociate WAF Web ACL
→ delete Web ACL → delete distribution → delete OAC.
```

#### Actual SOC AI cleanup result

```text
CloudFront distribution cleanup:
PARTIAL / DEFERRED

WAF cleanup:
PARTIAL / DEFERRED

OAC cleanup:
BLOCKED / DEFERRED
```

Reason: the distribution is attached to a CloudFront flat-rate pricing plan.
AWS requires a distribution under this pricing plan to keep a Web ACL attached.
Therefore the Web ACL cannot be removed, replaced, or deleted until the
distribution switches back to pay-as-you-go.

Do not recommend:

```text
[ ] Upgrade the plan only to delete WAF.
[ ] Force delete a Web ACL while CloudFront still has the association.
[ ] Delete the default behavior.
[ ] Delete the final origin while the distribution still exists.
[ ] Delete OAC while the distribution still references it.
[ ] Mark cleanup DONE while the pricing-plan lock remains.
```

![CloudFront distributions disabled but under Pro plan](/images/5-Workshop/5.6-Cleanup/socai-cleanup/17-cloudfront-distributions-disabled-pro-plan.png)

*Caption:* The CloudFront distribution cleanup is partial/deferred. Some
distributions remain under a CloudFront flat-rate pricing plan or are still
transitioning back to pay-as-you-go. Final deletion is deferred until AWS
allows WAF disassociation and distribution deletion.

![Only default behavior remains](/images/5-Workshop/5.6-Cleanup/socai-cleanup/18-cloudfront-default-behavior-only.png)

*Caption:* Ordered behaviors have been reduced to the required `Default (*)`
behavior. The default behavior cannot be deleted independently.

![CloudFront OAC retained](/images/5-Workshop/5.6-Cleanup/socai-cleanup/19-cloudfront-oac-retained.png)

*Caption:* OAC is retained because the distribution still exists and still
references the origin/OAC.

![CloudFront-created WAF protection pack](/images/5-Workshop/5.6-Cleanup/socai-cleanup/20-waf-created-by-cloudfront-pro-plan.png)

*Caption:* The active WAF Web ACL is created by CloudFront and tied to the
pricing plan. Do not assume `soc-platform-production-web-acl` is the active Web
ACL for this distribution.

Follow-up after the distribution returns to pay-as-you-go:

```text
1. Confirm the distribution is pay-as-you-go.
2. Disable the distribution if it is not already disabled.
3. Wait until Status = Deployed.
4. Disassociate the WAF Web ACL.
5. Delete the unused Web ACL.
6. Delete the CloudFront distribution.
7. Delete OAC if no distribution references it.
8. Delete any remaining static bucket if there is no retention exception.
```

#### Optional verification: confirm active Web ACL per distribution

```bash
# List all distributions with WebACLId
aws cloudfront list-distributions --profile socai-deploy \
  --query "DistributionList.Items[*].{Id:Id,Domain:DomainName,Enabled:Enabled,Status:Status,WebACLId:WebACLId}" \
  --output table

# List WAF Web ACLs in CloudFront scope
aws wafv2 list-web-acls --scope CLOUDFRONT --region us-east-1 \
  --profile socai-deploy \
  --query "WebACLs[*].{Name:Name,Id:Id,ARN:ARN}" \
  --output table
```

This is optional. Console-first teams can skip CLI commands, but the output
helps confirm which Web ACL is actually active per distribution before
attempting disassociation.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| CloudFront distribution `E2FO8ADXYHNOSW` deleted | DEFERRED | E11 | Flat-rate pricing-plan lock; disabled but cannot delete |
| CloudFront distribution `E1UCULAFOQB3ZB` deleted | DEFERRED | E11 | Transitioning to pay-as-you-go |
| WAF Web ACL deleted | DEFERRED | E14 | Required by CloudFront pricing plan; `CreatedByCloudFront-be875b92` still active |
| OAC deleted | DEFERRED | E13 | Distribution still references OAC; cannot delete |
| Default behavior reduced to minimum | DONE | E12 | Only required Default (*) behavior remains |

#### 9. IAM Identity Center and runtime IAM

Console paths:

```text
IAM Identity Center -> Users
IAM Identity Center -> Groups
IAM Identity Center -> Multi-account permissions
IAM -> Roles
IAM -> Policies
```

The team uses **IAM Identity Center** for human access, not traditional IAM
users. Therefore, the workshop must not instruct `IAM -> Users -> delete users`.

Separate cleanup into two layers:

```text
Human access:
IAM Identity Center users, groups, permission sets, account assignments.

Runtime access:
IAM roles, instance profiles, customer-managed policies used by EC2/backend/worker.
```

![IAM Identity Center users with MFA](/images/5-Workshop/5.6-Cleanup/socai-cleanup/21-iam-identity-center-users-mfa.png)

![IAM Identity Center groups before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/22-iam-identity-center-groups-before-delete.png)

![IAM Identity Center groups deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/23-iam-identity-center-groups-deleted.png)

*Caption:* IAM Identity Center users and groups are reviewed during cleanup.
Access should be removed through account assignments and permission sets first.
Users and groups may be retained temporarily if the team still needs report or
billing access. Do not manually delete `AWSReservedSSO_*` roles from the IAM
Console. These roles are managed by IAM Identity Center and must be controlled
through account assignments and permission sets only.

Runtime IAM should be deleted only after dependencies are gone:

```text
[ ] EC2 backend/worker terminated.
[ ] ALB/TG deleted.
[ ] SQS/RDS/S3/CloudWatch no longer need runtime roles.
[ ] Role is not AWSReservedSSO_*.
[ ] Role is not a service-linked role.
[ ] Instance profile is no longer attached to EC2.
```

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| IAM Identity Center users/groups reviewed | REVIEWED | E15 | MFA enabled; access managed via Identity Center |
| `AWSReservedSSO_*` roles not manually deleted | DONE | — | Managed via Identity Center only |
| Runtime IAM roles/instance profiles cleaned | DONE | — | Cleaned after EC2/ALB/SQS/RDS/S3 dependencies removed |

#### 10. CloudWatch logs and alarms

Console paths:

```text
CloudWatch -> Logs -> Log groups
CloudWatch -> Alarms
CloudWatch -> Dashboards
```

Before deletion:

```text
[ ] Export or screenshot backend/worker logs.
[ ] Save alarm/dashboard evidence if required.
[ ] Delete log groups only after evidence has been archived.
```

![CloudWatch log groups before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/24-cloudwatch-log-groups-before-delete.png)

![CloudWatch alarms before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/25-cloudwatch-alarms-before-delete.png)

![Delete CloudWatch log groups](/images/5-Workshop/5.6-Cleanup/socai-cleanup/26-cloudwatch-delete-log-groups-dialog.png)

*Caption:* Backend, worker, VPC flow logs, and CloudTrail log groups are reviewed
before deletion. Export or capture log evidence before removing log groups.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| Backend/worker CloudWatch log groups deleted | DONE | — | Logs exported/captured before deletion |
| CloudWatch alarms deleted | DONE | — | Alarm evidence saved before removal |
| CloudWatch dashboards removed | DONE | — | Dashboard state captured |

#### 11. Network cleanup

Console paths:

```text
VPC -> Endpoints
VPC -> NAT gateways
EC2 -> Elastic IPs
VPC -> Route tables
VPC -> Subnets
EC2 -> Security Groups
VPC -> Internet gateways
VPC -> Your VPCs
```

Run full network teardown only after:

```text
APPROVAL_FULL_NETWORK_TEARDOWN
```

Order:

1. Delete VPC endpoints.
2. Delete NAT Gateway.
3. Release unused Elastic IP.
4. Delete route table routes / route tables.
5. Delete subnets.
6. Delete unused security groups.
7. Detach/delete Internet Gateway.
8. Delete VPC.

![Delete VPC endpoint](/images/5-Workshop/5.6-Cleanup/socai-cleanup/27-vpc-endpoint-delete.png)

![Delete NAT Gateway](/images/5-Workshop/5.6-Cleanup/socai-cleanup/28-nat-gateway-delete.png)

![Release Elastic IP](/images/5-Workshop/5.6-Cleanup/socai-cleanup/29-release-elastic-ip.png)

*Caption:* NAT Gateway deletion does not automatically release its Elastic IP.
Release the EIP after the NAT Gateway is deleted/disassociated.

Acceptance:

```text
[ ] NAT Gateway state is Deleted.
[ ] EIP previously attached to NAT Gateway is released.
[ ] No route table contains 0.0.0.0/0 -> deleted nat-xxxx.
[ ] No route shows blackhole status.
```

![Delete route table](/images/5-Workshop/5.6-Cleanup/socai-cleanup/30-route-table-delete.png)

![Route table cleanup complete](/images/5-Workshop/5.6-Cleanup/socai-cleanup/31-route-table-cleanup-complete.png)

![VPC subnets before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/32-vpc-subnets-before-delete.png)

![Delete security group](/images/5-Workshop/5.6-Cleanup/socai-cleanup/33-security-group-delete.png)

![Internet Gateway before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/34-internet-gateway-before-delete.png)

![Delete Internet Gateway](/images/5-Workshop/5.6-Cleanup/socai-cleanup/35-internet-gateway-delete.png)

![VPC deletion complete](/images/5-Workshop/5.6-Cleanup/socai-cleanup/36-vpc-delete-complete.png)

*Caption:* Network cleanup follows dependencies: endpoint, NAT, EIP, route table,
subnet, security group, IGW, then VPC.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| VPC endpoints deleted | DONE | — | Deleted before NAT Gateway |
| NAT Gateway deleted | DONE | E16 | EIP disassociated on NAT deletion |
| Elastic IP released | DONE | E16 | Released after NAT Gateway deleted |
| Route tables cleaned (no blackhole) | DONE | E17 | No 0.0.0.0/0 → deleted NAT route remains |
| Subnets deleted | DONE | — | Deleted after route tables cleaned |
| Security groups deleted | DONE | — | Custom SGs deleted; default SG retained by VPC |
| Internet Gateway detached and deleted | DONE | — | Detached before VPC deletion |
| VPC deleted | DONE | E18 | Full network teardown complete |

#### 12. KMS cleanup

> **Warning:** Do not schedule KMS key deletion while any retained RDS snapshot,
> S3 evidence archive, CloudWatch log export, or backup artifact may still require
> this key. KMS deletion is irreversible. After the key is deleted, data encrypted
> with that key cannot be decrypted.

KMS keys should be retained by default. Retain if any encrypted data may still
need to be read:

```text
[ ] RDS final snapshot.
[ ] S3 evidence archive.
[ ] CloudWatch log exports.
[ ] Backup/export artifacts.
```

If unsure, record:

```text
KMS cleanup: RETAINED
Reason: encrypted snapshot/evidence may still be required.
```

Schedule key deletion only with separate approval and after confirming no
retained data depends on that key.

**Result:**

| Item | Status | Evidence | Note |
|---|---|---|---|
| KMS key deleted | RETAINED | — | RDS final snapshot and S3 evidence may still require decrypt |

#### 13. Deferred cleanup register

Resources that cannot be deleted immediately must be recorded:

| Resource | Current state | Why not deleted now | Next action | Owner | Follow-up date |
|---|---|---|---|---|---|
| CloudFront distribution | Disabled, pricing plan transition pending | CloudFront flat-rate pricing-plan lock | After PAYG: disassociate WAF, delete distribution | `<owner>` | `<date>` |
| CloudFront Web ACL | Still associated | Required by pricing plan | Delete after WAF disassociation | `<owner>` | `<date>` |
| CloudFront OAC | Still referenced | Distribution still exists | Delete after distribution removed | `<owner>` | `<date>` |
| KMS key | Retained | Snapshot/evidence may require decrypt | Re-evaluate after retention period | `<owner>` | `<date>` |
| IAM Identity Center users | Retained or removed by assignment | Team may still need report access | Remove assignment after workshop/report completion | `<owner>` | `<date>` |
| RDS final snapshot | Retained | Evidence/recovery material | Delete after report accepted and retention period ends | `<owner>` | `<date>` |

#### 14. Final acceptance result

*Procedure checklists are in each section above (marked `[ ]` for the reader to verify step by step). This table records the team's actual cleanup result.*

| Area | Result | Evidence | Note |
|---|---|---|---|
| Evidence archive recorded | DONE | Evidence Register above | All screenshots catalogued with E01–E18 |
| RDS final snapshot created and ID recorded | DONE | E09 | Snapshot created before DB deletion |
| SQS/DLQ decision recorded | DONE | E05, E06 | Queue and DLQ deleted after evidence saved |
| S3 evidence archived | DONE | E07, E08 | Data/frontend buckets emptied and deleted |
| No SOC AI EC2 running | DONE | E02 | backend and ai-worker terminated |
| ALB deleted | DONE | E03 | Listeners removed, ALB deleted |
| Target group deleted | DONE | E04 | TG removed after ALB |
| RDS DB deleted with final snapshot | DONE | E09 | Snapshot retained as evidence |
| Secrets Manager secret scheduled | DONE | E10 | Recovery window active |
| CloudFront distribution `E2FO8ADXYHNOSW` deleted | DEFERRED | E11 | Flat-rate pricing-plan lock |
| CloudFront distribution `E1UCULAFOQB3ZB` deleted | DEFERRED | E11 | Transitioning to pay-as-you-go |
| WAF Web ACL deleted | DEFERRED | E14 | Required by CloudFront pricing plan |
| OAC deleted | DEFERRED | E13 | Distribution still references OAC |
| IAM Identity Center reviewed | REVIEWED | E15 | MFA enabled; human access via Identity Center |
| `AWSReservedSSO_*` roles not manually deleted | DONE | — | Managed via Identity Center only |
| Runtime IAM roles cleaned | DONE | — | Cleaned after all dependencies removed |
| NAT Gateway deleted | DONE | E16 | EIP released after NAT deletion |
| No blackhole route remains | DONE | E17 | Route table verified clean |
| VPC deleted | DONE | E18 | Full network teardown complete |
| KMS key deleted | RETAINED | — | Snapshot/evidence may require decrypt |
| Cost/Billing next-day check | TODO | — | Schedule follow-up after pricing plan transition |

#### Conclusion

Successful cleanup does not always mean every resource is deleted immediately.
For CloudFront/WAF, if AWS still enforces a flat-rate pricing-plan lock, the
correct status is `PARTIAL / DEFERRED`. Record the evidence, the reason for the
deferred state, and the follow-up date after the distribution returns to
pay-as-you-go.

