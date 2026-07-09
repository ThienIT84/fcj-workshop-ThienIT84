---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Dọn dẹp tài nguyên

Phần này hướng dẫn dọn dẹp môi trường SOC AI sau khi hoàn tất workshop. Nội dung
được viết theo hướng **AWS Console first**: thao tác trên dashboard AWS, lưu
evidence bằng screenshot, sau đó mới xóa tài nguyên theo đúng thứ tự phụ thuộc.

> **Lưu ý trước khi publish:** các screenshot trong phần này có thể chứa account
> ID, tên user, ARN, domain, IP public hoặc tên resource thật. Hãy mask các thông
> tin nhạy cảm trước khi đưa lên site public.

#### Mục tiêu cleanup

+ Lưu lại evidence cần nộp trước khi xóa tài nguyên.
+ Xóa tài nguyên theo thứ tự dependency để tránh lỗi.
+ Tránh phát sinh chi phí từ EC2, ALB, NAT Gateway, RDS, SQS, S3, CloudFront và
  CloudWatch.
+ Ghi rõ tài nguyên nào đã xóa, tài nguyên nào được giữ lại, bị block hoặc
  deferred.
+ Không cố xóa tài nguyên khi AWS đang khóa bởi billing/pricing-plan constraint.

#### Nguyên tắc an toàn

Trước khi xóa bất kỳ tài nguyên nào, nhóm cần xác nhận:

```text
[ ] Đúng AWS account.
[ ] Đúng region ap-southeast-1 cho tài nguyên regional.
[ ] CloudFront/WAF được kiểm tra ở Global/CloudFront scope.
[ ] Evidence đã được archive ngoài tài nguyên sắp xóa.
[ ] Không còn producer, replay script hoặc shipper gửi event mới.
[ ] Approval gate tương ứng đã được ghi lại nếu thao tác có rủi ro mất dữ liệu.
```

Approval gates khuyến nghị:

```text
APPROVAL_DELETE_RDS_WITH_FINAL_SNAPSHOT
APPROVAL_DELETE_S3_DATA_BUCKET_AFTER_EVIDENCE_BACKUP
APPROVAL_PURGE_OR_DELETE_DLQ_AFTER_FORENSIC_EVIDENCE
APPROVAL_FULL_NETWORK_TEARDOWN
```

#### Trạng thái cleanup tổng quan

| Hạng mục | Trạng thái | Ghi chú |
|---|---|---|
| EC2 backend/worker | `DELETED` | Terminate sau khi dừng service và lưu log/evidence. |
| ALB / Target Group | `DELETED` | Xóa sau khi compute không còn cần nhận traffic. |
| SQS main queue / DLQ | `DELETED` | Chỉ xóa sau khi đã lưu queue/DLQ evidence. |
| S3 data/frontend buckets | `DELETED` hoặc `RETAINED` | Chỉ xóa data bucket sau khi archive evidence/model artifacts. |
| RDS PostgreSQL | `DELETED WITH FINAL SNAPSHOT` | Cần export/chụp `final_alerts` trước khi delete. |
| Secrets Manager | `SCHEDULED DELETION` | Dùng recovery window, không force delete. |
| CloudFront distribution | `PARTIAL / DEFERRED` | Bị CloudFront flat-rate pricing plan lock. |
| CloudFront WAF Web ACL | `PARTIAL / DEFERRED` | Web ACL vẫn bắt buộc khi distribution còn pricing plan. |
| CloudFront OAC | `BLOCKED / DEFERRED` | Không xóa OAC khi distribution còn reference. |
| IAM Identity Center | `REVIEWED` | Không xóa IAM users truyền thống; quản lý bằng Identity Center. |
| NAT/EIP/VPC | `DELETED` hoặc `RETAINED` | Full network teardown cần approval riêng. |
| KMS | `RETAINED` mặc định | Chỉ schedule deletion nếu chắc không còn snapshot/evidence cần decrypt. |

#### 1. Lưu evidence trước khi cleanup

Trước khi dừng dịch vụ hoặc xóa tài nguyên, lưu tối thiểu:

```text
[ ] Dashboard hoặc /api/alerts/latest evidence.
[ ] D5-5 idempotency PASS evidence.
[ ] RDS final_alerts query/export.
[ ] SQS/DLQ queue state.
[ ] S3 evidence/model artifact inventory.
[ ] CloudWatch backend/worker logs.
[ ] CloudFront/WAF/OAC state.
[ ] IAM Identity Center users/groups/assignments.
[ ] Billing hoặc pricing-plan state nếu CloudFront bị lock.
```

Ví dụ evidence RDS `final_alerts` trước khi xóa database:

![RDS final_alerts evidence](/images/5-Workshop/5.6-Cleanup/socai-cleanup/10-rds-final-alerts-evidence.png)

*Caption:* Bảng `final_alerts` vẫn truy xuất được trước cleanup, bao gồm các
alert SQL Injection và Cross-Site Scripting đã dùng trong demo.

#### 2. Dừng backend, worker và EC2

Console path:

```text
EC2 -> Instances
```

Thao tác:

1. Dừng producer/replay/tailer đang gửi event.
2. Dừng worker sau khi SQS main queue đã được drain hoặc đã có quyết định lưu
   queue evidence.
3. Dừng backend khi không còn cần nhận event mới.
4. Terminate EC2 backend và worker sau khi log/evidence đã lưu.

![Terminate backend and worker instances](/images/5-Workshop/5.6-Cleanup/socai-cleanup/01-ec2-terminate-backend-worker.png)

*Caption:* Hai EC2 instance `backend` và `ai-worker` được chọn để terminate sau
khi evidence và queue state đã được xử lý.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| backend EC2 đã terminate | DONE | E02 | Instance terminated sau khi lưu log |
| ai-worker EC2 đã terminate | DONE | E02 | Instance terminated sau khi xử lý queue state |

#### 3. Xóa ALB và Target Group

Console path:

```text
EC2 -> Load Balancers
EC2 -> Target Groups
```

Thứ tự:

1. Xác nhận không còn request mới cần route qua ALB.
2. Xóa hoặc detach listener/rule nếu còn.
3. Delete ALB.
4. Delete target group sau khi không còn load balancer reference.

![Delete ALB action](/images/5-Workshop/5.6-Cleanup/socai-cleanup/02-alb-delete-menu.png)

![Confirm ALB deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/03-alb-delete-confirm.png)

![Delete target group](/images/5-Workshop/5.6-Cleanup/socai-cleanup/04-target-group-delete-menu.png)

*Caption:* ALB `socai-dev-alb` và target group `socai-dev-backend-tg` được xóa
sau khi compute không còn cần nhận traffic.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| ALB `socai-dev-alb` đã xóa | DONE | E03 | Listeners removed, ALB deleted |
| Target group `socai-dev-backend-tg` đã xóa | DONE | E04 | TG xóa sau ALB |

#### 4. Xóa SQS main queue và DLQ

Console path:

```text
Amazon SQS -> Queues
```

Thao tác:

1. Lưu screenshot queue attributes và số lượng message.
2. Inspect DLQ nếu còn message.
3. Chỉ purge/delete DLQ khi forensic evidence đã lưu và có approval.
4. Delete main queue.
5. Delete DLQ.

> `PurgeQueue` là irreversible. Không purge main queue hoặc DLQ nếu chưa lưu
> forensic evidence và chưa có approval.

Nếu queue sẽ bị delete hoàn toàn, thường không cần purge trước. Purge chỉ cần
thiết khi muốn giữ queue nhưng cần xóa toàn bộ message. Trong mọi trường hợp,
phải inspect và lưu DLQ evidence trước bất kỳ thao tác purge hoặc delete nào.

![Main SQS queue deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/05-sqs-main-queue-deleted.png)

![DLQ deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/06-sqs-dlq-deleted.png)

*Caption:* Main queue được xóa trước, sau đó DLQ được xóa sau khi đã hoàn tất
queue evidence và quyết định xử lý DLQ.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| SQS main queue đã xóa | DONE | E05 | Xóa sau khi lưu queue evidence |
| SQS DLQ đã xóa | DONE | E06 | Xóa sau forensic decision và approval |

#### 5. Archive và xóa S3 buckets

Console path:

```text
S3 -> Buckets
```

Data/artifact bucket phải được xử lý rất cẩn thận vì có thể chứa evidence,
model artifacts, D5-5 receipts, report screenshots và export files.

Checklist:

```text
[ ] Evidence/model artifacts đã được archive ngoài bucket.
[ ] Current objects đã được empty.
[ ] Nếu versioning enabled: xóa object versions.
[ ] Xóa delete markers.
[ ] Confirm bucket rỗng.
[ ] Delete bucket.
```

![S3 data bucket content before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/07-s3-data-bucket-contents.png)

![S3 delete objects status](/images/5-Workshop/5.6-Cleanup/socai-cleanup/08-s3-delete-objects-status.png)

![S3 buckets deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/09-s3-buckets-deleted.png)

*Caption:* Data bucket được kiểm tra trước khi xóa. Sau đó objects được xóa
thành công và bucket frontend/static được delete khỏi danh sách buckets.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| S3 data/artifact bucket đã xóa | DONE | E07, E08 | Objects, versions, delete markers đã xóa |
| S3 frontend/static bucket đã xóa | DONE | E08 | Bucket removed from list |

#### 6. Xóa RDS với final snapshot

Console path:

```text
RDS -> Databases
```

Gate bắt buộc:

```text
[ ] APPROVAL_DELETE_RDS_WITH_FINAL_SNAPSHOT recorded.
[ ] final_alerts evidence đã lưu.
[ ] Deletion protection đã kiểm tra.
[ ] Create final snapshot được bật.
[ ] Final snapshot name có timestamp hoặc tên rõ ràng.
```

![Delete RDS with final snapshot](/images/5-Workshop/5.6-Cleanup/socai-cleanup/11-rds-delete-final-snapshot.png)

![RDS delete confirmation](/images/5-Workshop/5.6-Cleanup/socai-cleanup/12-rds-delete-confirmation.png)

![RDS deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/13-rds-deleted.png)

*Caption:* RDS `socai-dev-postgres-01` được xóa với final snapshot. Sau khi
delete, RDS console không còn DB instance của workshop.

> **Lưu ý chi phí:** Xóa DB instance sẽ dừng chi phí runtime của instance.
> Tuy nhiên, retained automated backups và manual snapshots vẫn có thể phát sinh
> storage charges cho đến khi được xóa. Final snapshot được giữ lại có chủ đích
> làm evidence và recovery material. Review và xóa sau khi thời hạn giữ
> báo cáo/workshop kết thúc.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| RDS `socai-dev-postgres-01` DB instance đã xóa | DONE | E09 | Xóa kèm final snapshot |
| Final snapshot được tạo và giữ lại | RETAINED | E09 | Evidence/recovery material — xóa sau retention period |

#### 7. Schedule deletion cho Secrets Manager

Console path:

```text
Secrets Manager -> Secrets
```

Thao tác:

1. Mở secret metadata.
2. Không chụp hoặc reveal secret value.
3. Chỉ delete/schedule deletion sau khi backend/worker đã dừng và RDS snapshot
   đã ổn.
4. Dùng recovery window, không dùng force delete.

![Secrets Manager list before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/14-secrets-manager-list.png)

![Secret metadata before delete](/images/5-Workshop/5.6-Cleanup/socai-cleanup/15-secret-before-delete.png)

![Secret scheduled for deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/16-secret-scheduled-deletion.png)

*Caption:* Secret `socai-dev/rds/app` được schedule deletion với recovery window.
Secret value không được hiển thị trong evidence. Secret được schedule deletion sẽ
bị inaccessible ngay lập tức nhưng có thể recover trước khi hết recovery window.
Secret đang được schedule deletion không phát sinh thêm charges.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| Secret `socai-dev/rds/app` được schedule deletion | DONE | E10 | Recovery window active — inaccessible, value không hiển thị |

#### 8. CloudFront, OAC và WAF: cleanup deferred

Phần này ghi lại cả quy trình cleanup CloudFront chuẩn lẫn kết quả thực tế của
SOC AI. Trong workshop này, việc xóa hoàn toàn bị deferred vì distribution vẫn
đang bị khóa bởi CloudFront flat-rate pricing plan.

Console paths:

```text
CloudFront -> Distributions
CloudFront -> Security -> Origin access
WAF & Shield -> Protection packs (web ACLs)
```

#### CloudFront inventory found during cleanup

| Distribution ID | Domain | Mục đích | Active Web ACL | Cleanup status |
|---|---|---|---|---|
| E2FO8ADXYHNOSW | dj9itpaxbbdgy.cloudfront.net | SOC AI dev CloudFront | CreatedByCloudFront-be875b92 | Deferred – flat-rate plan lock |
| E1UCULAFOQB3ZB | d2s3u1yztzei92.cloudfront.net | soc-platform-frontend | CloudFront-created Web ACL | Đang chuyển về pay-as-you-go |
| soc-platform-production-web-acl | N/A | Custom WAF (không active) | Không gắn với distribution đã test | Không giả định nó đang bảo vệ CloudFront active |

> `soc-platform-production-web-acl` **không phải** Web ACL active của E2FO8ADXYHNOSW.
> Web ACL thực sự đang dùng là `CreatedByCloudFront-be875b92`. Không thêm WAF rule
> vào `soc-platform-production-web-acl` để affect distribution này.

#### Quy trình cleanup chuẩn (Expected normal cleanup path)

```text
Disable distribution → đợi Status = Deployed → disassociate WAF Web ACL
→ delete Web ACL → delete distribution → delete OAC.
```

#### Kết quả thực tế SOC AI (Actual SOC AI cleanup result)

```text
CloudFront distribution cleanup:
PARTIAL / DEFERRED

WAF cleanup:
PARTIAL / DEFERRED

OAC cleanup:
BLOCKED / DEFERRED
```

Lý do: distribution đang gắn CloudFront flat-rate pricing plan. AWS yêu cầu
distribution trong pricing plan phải có Web ACL đi kèm. Vì vậy không thể remove,
replace hoặc delete Web ACL cho tới khi distribution chuyển về pay-as-you-go.

Không được hướng dẫn:

```text
[ ] Upgrade plan chỉ để xóa WAF.
[ ] Force delete Web ACL khi CloudFront còn association.
[ ] Delete default behavior.
[ ] Delete final origin khi distribution vẫn tồn tại.
[ ] Delete OAC khi distribution vẫn reference OAC.
[ ] Ghi cleanup DONE nếu còn pricing-plan lock.
```

![CloudFront distributions disabled but under Pro plan](/images/5-Workshop/5.6-Cleanup/socai-cleanup/17-cloudfront-distributions-disabled-pro-plan.png)

*Caption:* Cleanup CloudFront distribution là partial/deferred. Một số distribution
vẫn đang ở CloudFront flat-rate pricing plan hoặc đang trong quá trình chuyển về
pay-as-you-go. Việc xóa hoàn toàn bị deferred cho đến khi AWS cho phép WAF
disassociation và distribution deletion.

![Only default behavior remains](/images/5-Workshop/5.6-Cleanup/socai-cleanup/18-cloudfront-default-behavior-only.png)

*Caption:* Ordered behaviors đã được giảm còn required `Default (*)`. Default
behavior không thể xóa độc lập.

![CloudFront OAC retained](/images/5-Workshop/5.6-Cleanup/socai-cleanup/19-cloudfront-oac-retained.png)

*Caption:* OAC vẫn được giữ lại vì distribution còn tồn tại và vẫn reference
origin/OAC.

![CloudFront-created WAF protection pack](/images/5-Workshop/5.6-Cleanup/socai-cleanup/20-waf-created-by-cloudfront-pro-plan.png)

*Caption:* Active WAF Web ACL là Web ACL do CloudFront tạo, gắn với pricing
plan. Không giả định `soc-platform-production-web-acl` là Web ACL active của
distribution.

Follow-up sau khi distribution về pay-as-you-go:

```text
1. Confirm distribution đã pay-as-you-go.
2. Disable distribution nếu chưa disabled.
3. Đợi Status = Deployed.
4. Disassociate WAF Web ACL.
5. Delete unused Web ACL.
6. Delete CloudFront distribution.
7. Delete OAC nếu không còn distribution reference.
8. Delete static bucket còn lại nếu không có retention exception.
```

#### Xác minh tùy chọn: kiểm tra Web ACL active theo từng distribution

```bash
# Liệt kê tất cả distribution kèm WebACLId
aws cloudfront list-distributions --profile socai-deploy \
  --query "DistributionList.Items[*].{Id:Id,Domain:DomainName,Enabled:Enabled,Status:Status,WebACLId:WebACLId}" \
  --output table

# Liệt kê WAF Web ACL trong CloudFront scope
aws wafv2 list-web-acls --scope CLOUDFRONT --region us-east-1 \
  --profile socai-deploy \
  --query "WebACLs[*].{Name:Name,Id:Id,ARN:ARN}" \
  --output table
```

Lệnh CLI này là tùy chọn. Team Console-first có thể bỏ qua, nhưng output giúp
xác nhận Web ACL thực sự active theo từng distribution trước khi thực hiện
disassociation.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| CloudFront distribution `E2FO8ADXYHNOSW` đã xóa | DEFERRED | E11 | Flat-rate pricing-plan lock; disabled nhưng không thể xóa |
| CloudFront distribution `E1UCULAFOQB3ZB` đã xóa | DEFERRED | E11 | Đang chuyển về pay-as-you-go |
| WAF Web ACL đã xóa | DEFERRED | E14 | Bắt buộc bởi CloudFront pricing plan; `CreatedByCloudFront-be875b92` vẫn active |
| OAC đã xóa | DEFERRED | E13 | Distribution vẫn reference OAC |
| Default behavior đã giảm xuống tối thiểu | DONE | E12 | Chỉ còn required Default (*) behavior |

#### 9. IAM Identity Center và runtime IAM

Console paths:

```text
IAM Identity Center -> Users
IAM Identity Center -> Groups
IAM Identity Center -> Multi-account permissions
IAM -> Roles
IAM -> Policies
```

Nhóm sử dụng **IAM Identity Center** cho human access, không phải IAM users
truyền thống. Vì vậy workshop không hướng dẫn `IAM -> Users -> delete users`.

Tách cleanup thành hai lớp:

```text
Human access:
IAM Identity Center users, groups, permission sets, account assignments.

Runtime access:
IAM roles, instance profiles, customer-managed policies dùng bởi EC2/backend/worker.
```

![IAM Identity Center users with MFA](/images/5-Workshop/5.6-Cleanup/socai-cleanup/21-iam-identity-center-users-mfa.png)

![IAM Identity Center groups before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/22-iam-identity-center-groups-before-delete.png)

![IAM Identity Center groups deleted](/images/5-Workshop/5.6-Cleanup/socai-cleanup/23-iam-identity-center-groups-deleted.png)

*Caption:* IAM Identity Center users và groups được review trong quá trình cleanup.
Quyền truy cập nên được gỡ bỏ thông qua account assignments và permission sets trước.
Users và groups có thể được giữ lại tạm thời nếu team còn cần truy cập báo cáo hoặc
billing. Không xóa thủ công `AWSReservedSSO_*` roles trong IAM Console. Các role này
do IAM Identity Center quản lý và phải được kiểm soát qua account assignments và
permission sets mà thôi.

Runtime IAM chỉ xóa sau khi dependency đã gone:

```text
[ ] EC2 backend/worker terminated.
[ ] ALB/TG deleted.
[ ] SQS/RDS/S3/CloudWatch không còn cần runtime role.
[ ] Role không phải AWSReservedSSO_*.
[ ] Role không phải service-linked role.
[ ] Instance profile không còn attached với EC2.
```

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| IAM Identity Center users/groups đã review | REVIEWED | E15 | MFA enabled; quản lý qua Identity Center |
| `AWSReservedSSO_*` roles không bị xóa thủ công | DONE | — | Managed via Identity Center only |
| Runtime IAM roles/instance profiles đã xóa | DONE | — | Xóa sau khi tất cả dependency đã gone |

#### 10. CloudWatch logs và alarms

Console paths:

```text
CloudWatch -> Logs -> Log groups
CloudWatch -> Alarms
CloudWatch -> Dashboards
```

Trước khi xóa:

```text
[ ] Export hoặc screenshot backend/worker logs.
[ ] Lưu alarm/dashboard evidence nếu cần.
[ ] Chỉ delete log groups sau khi evidence đã archive.
```

![CloudWatch log groups before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/24-cloudwatch-log-groups-before-delete.png)

![CloudWatch alarms before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/25-cloudwatch-alarms-before-delete.png)

![Delete CloudWatch log groups](/images/5-Workshop/5.6-Cleanup/socai-cleanup/26-cloudwatch-delete-log-groups-dialog.png)

*Caption:* Backend, worker, VPC flow logs và CloudTrail log groups được review
trước khi delete. Log evidence nên được export hoặc chụp trước khi xóa.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| Backend/worker CloudWatch log groups đã xóa | DONE | — | Logs đã export/capture trước khi xóa |
| CloudWatch alarms đã xóa | DONE | — | Alarm evidence đã lưu |
| CloudWatch dashboards đã xóa | DONE | — | Dashboard state đã capture |

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

Chỉ full network teardown sau khi đã có:

```text
APPROVAL_FULL_NETWORK_TEARDOWN
```

Thứ tự:

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

*Caption:* NAT Gateway deletion không tự release Elastic IP. Cần release EIP
sau khi NAT Gateway deleted/disassociated.

Acceptance:

```text
[ ] NAT Gateway state là Deleted.
[ ] EIP đã gắn với NAT Gateway trước đó đã được release.
[ ] Không còn route table nào chứa 0.0.0.0/0 -> deleted nat-xxxx.
[ ] Không có route nào ở trạng thái blackhole.
```

![Delete route table](/images/5-Workshop/5.6-Cleanup/socai-cleanup/30-route-table-delete.png)

![Route table cleanup complete](/images/5-Workshop/5.6-Cleanup/socai-cleanup/31-route-table-cleanup-complete.png)

![VPC subnets before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/32-vpc-subnets-before-delete.png)

![Delete security group](/images/5-Workshop/5.6-Cleanup/socai-cleanup/33-security-group-delete.png)

![Internet Gateway before deletion](/images/5-Workshop/5.6-Cleanup/socai-cleanup/34-internet-gateway-before-delete.png)

![Delete Internet Gateway](/images/5-Workshop/5.6-Cleanup/socai-cleanup/35-internet-gateway-delete.png)

![VPC deletion complete](/images/5-Workshop/5.6-Cleanup/socai-cleanup/36-vpc-delete-complete.png)

*Caption:* Network cleanup đi theo dependency: endpoint, NAT, EIP, route table,
subnet, security group, IGW, rồi mới xóa VPC.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| VPC endpoints đã xóa | DONE | — | Xóa trước NAT Gateway |
| NAT Gateway đã xóa | DONE | E16 | EIP disassociated khi NAT xóa |
| Elastic IP đã release | DONE | E16 | Release sau khi NAT Gateway deleted |
| Route tables đã clean (không còn blackhole) | DONE | E17 | Không còn 0.0.0.0/0 → deleted NAT route |
| Subnets đã xóa | DONE | — | Xóa sau route tables |
| Security groups đã xóa | DONE | — | Custom SG xóa; default SG retained by VPC |
| Internet Gateway đã detach và xóa | DONE | — | Detach trước khi xóa VPC |
| VPC đã xóa | DONE | E18 | Full network teardown hoàn tất |

#### 12. KMS cleanup

> **Cảnh báo:** Không schedule xóa KMS key trong khi bất kỳ RDS snapshot, S3 evidence
> archive, CloudWatch log export hoặc backup artifact nào vẫn còn cần key này. Xóa KMS
> là irreversible. Sau khi key bị xóa, dữ liệu được mã hóa bằng key đó không thể
> decrypt được nữa.

KMS key nên được retain mặc định. Retain nếu bất kỳ dữ liệu encrypted nào còn
cần đọc lại:

```text
[ ] RDS final snapshot.
[ ] S3 evidence archive.
[ ] CloudWatch log exports.
[ ] Backup/export artifacts.
```

Nếu chưa chắc, ghi:

```text
KMS cleanup: RETAINED
Reason: encrypted snapshot/evidence may still be required.
```

Chỉ schedule key deletion khi có approval riêng và chắc chắn không còn dữ liệu
phụ thuộc key đó.

**Kết quả:**

| Item | Status | Evidence | Ghi chú |
|---|---|---|---|
| KMS key đã xóa | RETAINED | — | RDS snapshot và S3 evidence có thể vẫn cần decrypt |

#### 13. Deferred cleanup register

Các tài nguyên chưa thể xóa ngay cần được ghi vào bảng deferred cleanup:

| Resource | Current state | Why not deleted now | Next action | Owner | Follow-up date |
|---|---|---|---|---|---|
| CloudFront distribution | Disabled, pricing plan transition pending | CloudFront flat-rate pricing-plan lock | Sau khi pay-as-you-go: disassociate WAF, delete distribution | `<owner>` | `<date>` |
| CloudFront Web ACL | Still associated | Required by pricing plan | Delete after WAF disassociation | `<owner>` | `<date>` |
| CloudFront OAC | Still referenced | Distribution still exists | Delete after distribution removed | `<owner>` | `<date>` |
| KMS key | Retained | Snapshot/evidence may require decrypt | Re-evaluate after retention period | `<owner>` | `<date>` |
| IAM Identity Center users | Retained or removed by assignment | Team may still need report access | Remove assignment after workshop/report done | `<owner>` | `<date>` |
| RDS final snapshot | Retained | Evidence/recovery material | Xóa sau khi báo cáo được chấp nhận và hết thời hạn retention | `<owner>` | `<date>` |

#### 14. Kết quả nghiệm thu cuối (Final acceptance result)

*Procedure checklist dùng `[ ]` ở từng section phía trên để người đọc tự kiểm tra từng bước. Bảng này ghi kết quả cleanup thực tế của nhóm.*

| Hạng mục | Kết quả | Evidence | Ghi chú |
|---|---|---|---|
| Evidence archive đã ghi lại | DONE | Evidence Register phía trên | Tất cả screenshot đã có trong E01–E18 |
| RDS final snapshot ID đã ghi lại | DONE | E09 | Snapshot tạo trước khi xóa DB |
| SQS/DLQ decision đã ghi lại | DONE | E05, E06 | Queue và DLQ đã xóa sau khi lưu evidence |
| S3 evidence đã archive | DONE | E07, E08 | Data/frontend buckets đã xóa |
| Không còn SOC AI EC2 chạy | DONE | E02 | backend và ai-worker đã terminate |
| ALB đã xóa | DONE | E03 | Listeners removed, ALB deleted |
| Target group đã xóa | DONE | E04 | TG xóa sau ALB |
| RDS DB đã xóa kèm final snapshot | DONE | E09 | Snapshot giữ lại làm evidence |
| Secrets Manager secret đã schedule | DONE | E10 | Recovery window active |
| CloudFront distribution `E2FO8ADXYHNOSW` đã xóa | DEFERRED | E11 | Flat-rate pricing-plan lock |
| CloudFront distribution `E1UCULAFOQB3ZB` đã xóa | DEFERRED | E11 | Đang chuyển về pay-as-you-go |
| WAF Web ACL đã xóa | DEFERRED | E14 | Bắt buộc bởi CloudFront pricing plan |
| OAC đã xóa | DEFERRED | E13 | Distribution vẫn reference OAC |
| IAM Identity Center đã review | REVIEWED | E15 | MFA enabled; human access qua Identity Center |
| `AWSReservedSSO_*` roles không bị xóa thủ công | DONE | — | Managed via Identity Center only |
| Runtime IAM roles đã xóa | DONE | — | Xóa sau khi tất cả dependency removed |
| NAT Gateway đã xóa | DONE | E16 | EIP đã release sau NAT |
| Không còn blackhole route | DONE | E17 | Route table đã kiểm tra sạch |
| VPC đã xóa | DONE | E18 | Full network teardown hoàn tất |
| KMS key đã xóa | RETAINED | — | Snapshot/evidence có thể vẫn cần decrypt |
| Kiểm tra Cost/Billing ngày hôm sau | TODO | — | Lên lịch follow-up sau khi pricing plan transition |

#### Kết luận

Cleanup thành công không luôn có nghĩa là mọi resource đều đã bị delete ngay.
Với CloudFront/WAF, nếu AWS đang khóa bởi flat-rate pricing plan, trạng thái đúng
là `PARTIAL / DEFERRED`. Nhóm cần ghi rõ evidence, lý do deferred và ngày
follow-up sau khi distribution quay về pay-as-you-go.

