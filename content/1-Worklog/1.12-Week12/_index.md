---
title: "Week 12 - Final Demo, Cleanup, Evidence Archive and Workshop Deployment"
date: 2026-07-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary focus | Final demo validation, evidence archive, AWS cleanup, and workshop publication |
| Demo evidence | Dashboard/API alert evidence from the SQS/RDS-backed pipeline |
| Cleanup mode | Archive evidence first, then remove AWS resources in dependency order |
| Deleted resources | S3 buckets, SQS queues, RDS with final snapshot workflow, and related runtime resources where approved |
| Deferred resources | CloudFront distribution, WAF Web ACL, and OAC because of CloudFront Pro / flat-rate pricing plan transition |
| Publication target | Hugo workshop site deployed through Vercel for FCAJ review |
| Week status | Final documentation and cleanup completed, with CloudFront follow-up explicitly deferred |

## Objective

The objective of Week 12 was to close the SOC AI workshop project responsibly. After the async SQS/RDS pipeline was validated in Week 11, the final week focused on preserving evidence, checking the dashboard/API demo, cleaning up AWS resources to control cost, and publishing the Hugo workshop report for FCAJ review.

This week does not introduce a new runtime feature. It records the final as-built state, the cleanup result, and the remaining deferred CloudFront/WAF/OAC follow-up.

## Final Demo Validation

The final demo evidence showed that the platform could display a persisted security alert after the backend pipeline had processed and stored it. The dashboard view showed a SQL Injection alert with critical severity, AI2B evidence, Fusion confidence, MITRE mapping, and source/destination context.

This validates the presentation layer for the final report, but it should not be interpreted as proof of worker-to-dashboard realtime WebSocket broadcast. The strongest backend evidence remains the Week 11 SQS/RDS/idempotency evidence, while Week 12 confirms that the final demo view and public report were prepared.

## Cleanup Strategy

Cleanup followed a conservative dependency order:

```text
Archive evidence
-> stop producers and services
-> delete compute and queues
-> delete or snapshot database resources
-> delete S3 buckets after evidence backup
-> schedule secret deletion
-> clean up network/log resources where approved
-> record CloudFront/WAF/OAC deferred state
```

The cleanup goal was not to force every resource to disappear immediately. The goal was to avoid unnecessary cost, preserve required evidence, and leave a clear follow-up register for anything blocked by AWS constraints.

## Daily Worklog

| Activity | Date | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- |
| Final dashboard/API evidence review | 06/07/2026 | Reviewed the final alert dashboard/API evidence from the deployed SQS/RDS-backed pipeline | SQL Injection alert evidence was available for the final report | Realtime WebSocket broadcast was not claimed without separate evidence | Archive evidence before cleanup |
| Evidence archive and masking checklist | 07/07/2026 | Organized screenshots, D5-5 receipts, RDS evidence, cleanup notes, and report assets | Evidence was separated from resources that would be deleted | Screenshots containing account IDs, usernames, ARNs, domains, or personal details must be masked before publication | Start resource cleanup |
| S3, SQS, and RDS cleanup evidence | 09/07/2026 | Deleted S3 buckets after evidence handling, deleted SQS queues, and started RDS deletion with final snapshot workflow | Major cost-generating data/queue resources were removed or placed into deletion flow | RDS deletion must keep final snapshot evidence; queue/DLQ deletion must happen after forensic decision | Record deferred CloudFront state |
| CloudFront/WAF/OAC deferred state | 10/07/2026 | Checked CloudFront distribution, WAF Web ACL, and OAC status after cleanup | CloudFront cleanup was partial/deferred because the distribution was under a Pro / flat-rate pricing plan and changing back to pay-as-you-go | Do not claim CloudFront, WAF, or OAC deletion until AWS allows disassociation and deletion | Prepare Hugo publication |
| Hugo build and deployment preparation | 11/07/2026 | Reviewed Hugo worklog, cleanup page, image paths, and Vercel deployment runbook | The workshop site was ready for build/deploy validation | Public pages must not expose raw or unmasked evidence | Verify public Vercel site |
| Vercel publication and final package | 12/07/2026 | Verified the public Vercel site and prepared the final submission package for FCAJ | The workshop report became accessible through a public review URL | Personal information on the landing page should be reviewed before final sharing | Submit final report link and evidence summary |

## Evidence Gallery

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e01-final-dashboard-alert-ui.png" alt="Final dashboard SQL Injection alert">
  <figcaption>Figure 1 - The final dashboard showed a persisted Critical SQL Injection alert with AI2B, Fusion, MITRE, and network context.</figcaption>
</figure>

**Observation:** This validates the displayed final alert state. It is not used to claim that the worker broadcast the alert to the dashboard through a realtime WebSocket channel.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e02-s3-data-bucket-deleted.png" alt="S3 data bucket object cleanup evidence">
  <figcaption>Figure 2 - The S3 data bucket objects were deleted before bucket removal after evidence and artifact handling.</figcaption>
</figure>

**Observation:** Data/artifact buckets must be cleaned only after evidence, model artifacts, receipts, and screenshots are archived outside the bucket.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e04-rds-final-snapshot-confirmation.png" alt="RDS final snapshot confirmation evidence">
  <figcaption>Figure 3 - The RDS delete dialog kept final snapshot creation enabled before deleting the PostgreSQL instance.</figcaption>
</figure>

**Observation:** RDS deletion should preserve a final snapshot unless there is a separate written approval to skip it. This protects the final alert data after the live database is removed.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e05-sqs-queues-deleted.png" alt="SQS queues deleted evidence">
  <figcaption>Figure 4 - The SQS queue list showed no queues after the main queue and DLQ were deleted.</figcaption>
</figure>

**Observation:** Queue deletion happened after queue state and DLQ evidence decisions were handled. Purging was not required when deleting the queues themselves.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e06-cloudfront-deferred-pro-plan.png" alt="CloudFront deferred cleanup due to Pro plan evidence">
  <figcaption>Figure 5 - CloudFront distributions remained disabled under a Pro / flat-rate pricing plan, so final deletion was deferred.</figcaption>
</figure>

**Observation:** This is the key cleanup limitation. The distribution, Web ACL, and OAC should remain in the deferred register until AWS allows WAF disassociation and distribution deletion.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e07-s3-static-bucket-deleted.png" alt="S3 static bucket deleted evidence">
  <figcaption>Figure 6 - The frontend/static S3 bucket deletion was confirmed after public hosting was no longer needed from AWS.</figcaption>
</figure>

**Observation:** Static frontend cleanup was separated from report publication. The final FCAJ report was served from the Hugo/Vercel path instead of the deleted AWS S3 frontend bucket.

**Evidence note:** The RDS action-menu screenshot and a separate Vercel live-site screenshot were not preserved as public image files before cleanup. The RDS final-snapshot dialog above records the deletion safeguard, and the final public URL is validated through the configured Vercel deployment; this page does not present missing screenshots as evidence.

## CloudFront Deferred Cleanup Note

CloudFront distribution cleanup was partial/deferred. The distribution remained under a CloudFront Pro / flat-rate pricing plan and was changing back to pay-as-you-go. WAF disassociation, distribution deletion, and OAC deletion were deferred until AWS allows the pricing-plan transition to complete.

The correct follow-up is:

```text
1. Confirm CloudFront has returned to pay-as-you-go.
2. Keep or set the distribution disabled.
3. Wait until CloudFront status is Deployed.
4. Disassociate the CloudFront-created Web ACL.
5. Delete the unused Web ACL.
6. Delete the distribution using the latest ETag.
7. Delete OAC only when no distribution references it.
```

The Week 12 cleanup result should therefore be recorded as:

| Resource | Cleanup result | Reason |
| --- | --- | --- |
| CloudFront distribution | Deferred | Pro / flat-rate pricing-plan transition |
| CloudFront-created WAF Web ACL | Deferred | Required while the pricing plan association remains |
| CloudFront OAC | Deferred | Distribution still references it |
| S3, SQS, RDS, Secrets | Handled according to evidence and approval gates | Cleanup actions were completed or scheduled where supported |

## Hugo and Vercel Publication

After AWS cleanup, the workshop report needed a stable public review URL. Hugo was used to build the static report, and Vercel was used as the public hosting layer for FCAJ review.

The publication checklist was:

```text
[ ] Hugo local build passes.
[ ] Worklog Week 11 and Week 12 pages render.
[ ] Cleanup page renders in EN and VI.
[ ] Image paths point to curated public-safe screenshots.
[ ] No raw evidence folders are published.
[ ] Vercel production URL opens correctly.
[ ] Language switch stays on the Vercel domain.
```

## Final Outcome and Lessons Learned

Week 12 closed the project with evidence-first cleanup and public report delivery. The SOC AI platform had enough evidence to show the final dashboard/API result, SQS/RDS persistence, idempotency behavior, and cleanup discipline.

The most important lesson was that cleanup is also engineering work. Some resources can be deleted immediately, some need final snapshots or recovery windows, and some must be deferred because AWS service rules or pricing-plan transitions block deletion. Recording that boundary honestly makes the final report stronger than pretending every resource was removed at once.
