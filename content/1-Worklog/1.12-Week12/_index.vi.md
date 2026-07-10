---
title: "Tuần 12 - Demo cuối, Cleanup, Lưu Evidence và Deploy Workshop"
date: 2026-07-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Trọng tâm chính | Xác thực demo cuối, lưu evidence, cleanup AWS và publish workshop |
| Demo evidence | Dashboard/API alert evidence từ pipeline có SQS/RDS phía sau |
| Chế độ cleanup | Archive evidence trước, sau đó xóa AWS resources theo dependency order |
| Tài nguyên đã xử lý | S3 buckets, SQS queues, RDS với final snapshot workflow, và runtime resources liên quan khi đã được duyệt |
| Tài nguyên deferred | CloudFront distribution, WAF Web ACL và OAC do CloudFront Pro / flat-rate pricing plan transition |
| Nơi publish | Hugo workshop site deploy qua Vercel để FCAJ review |
| Trạng thái tuần | Hoàn tất tài liệu cuối và cleanup, riêng CloudFront follow-up được ghi deferred rõ ràng |

## Mục tiêu

Mục tiêu của Tuần 12 là đóng dự án SOC AI một cách có trách nhiệm. Sau khi async SQS/RDS pipeline được xác thực ở Tuần 11, tuần cuối tập trung vào lưu evidence, kiểm tra dashboard/API demo, cleanup AWS để kiểm soát chi phí, và publish Hugo workshop report cho FCAJ review.

Tuần này không thêm runtime feature mới. Nội dung chính là ghi lại trạng thái as-built cuối cùng, kết quả cleanup, và phần follow-up còn deferred của CloudFront/WAF/OAC.

## Xác thực demo cuối

Final demo evidence cho thấy platform có thể hiển thị một security alert đã được xử lý và lưu lại. Dashboard view hiển thị SQL Injection alert với critical severity, AI2B evidence, Fusion confidence, MITRE mapping và source/destination context.

Evidence UI này hữu ích cho phần demo cuối, nhưng không được hiểu là bằng chứng cho worker-to-dashboard realtime WebSocket broadcast. Evidence backend mạnh nhất vẫn là SQS/RDS/idempotency ở Tuần 11, còn Tuần 12 xác nhận final demo view và public report đã được chuẩn bị.

## Chiến lược cleanup

Cleanup đi theo dependency order thận trọng:

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

Mục tiêu cleanup không phải là ép mọi tài nguyên biến mất ngay lập tức. Mục tiêu là tránh chi phí không cần thiết, giữ evidence cần thiết, và có follow-up register rõ ràng cho những tài nguyên bị AWS constraint chặn.

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Rà soát dashboard/API evidence cuối | 06/07/2026 | Kiểm tra final alert dashboard/API evidence từ pipeline đã deploy với SQS/RDS phía sau | SQL Injection alert evidence sẵn sàng cho final report | Không claim realtime WebSocket broadcast nếu không có evidence riêng | Archive evidence trước cleanup |
| Evidence archive và checklist masking | 07/07/2026 | Tổ chức screenshots, D5-5 receipts, RDS evidence, cleanup notes và report assets | Evidence được tách khỏi các resource sẽ bị xóa | Screenshot chứa account ID, username, ARN, domain hoặc thông tin cá nhân cần được mask trước khi publish | Bắt đầu cleanup resource |
| Cleanup S3, SQS và RDS evidence | 09/07/2026 | Xóa S3 buckets sau khi xử lý evidence, xóa SQS queues, và bắt đầu RDS deletion với final snapshot workflow | Các resource dữ liệu/queue có khả năng phát sinh chi phí đã được xóa hoặc đưa vào deletion flow | RDS deletion cần giữ final snapshot evidence; queue/DLQ deletion cần sau forensic decision | Ghi nhận trạng thái CloudFront deferred |
| Trạng thái deferred của CloudFront/WAF/OAC | 10/07/2026 | Kiểm tra CloudFront distribution, WAF Web ACL và OAC sau cleanup | CloudFront cleanup là partial/deferred vì distribution còn trong Pro / flat-rate pricing plan và đang chuyển về pay-as-you-go | Không claim CloudFront, WAF hoặc OAC deletion cho tới khi AWS cho phép disassociation và deletion | Chuẩn bị publish Hugo |
| Hugo build và chuẩn bị deploy | 11/07/2026 | Rà soát Hugo worklog, cleanup page, image paths và Vercel deployment runbook | Workshop site sẵn sàng để build/deploy validation | Public pages không được expose raw hoặc unmasked evidence | Xác minh Vercel public site |
| Vercel publication và final package | 12/07/2026 | Kiểm tra public Vercel site và chuẩn bị final submission package cho FCAJ | Workshop report có public review URL | Thông tin cá nhân trên landing page cần được rà soát trước khi chia sẻ chính thức | Nộp final report link và evidence summary |

## Bộ ảnh minh chứng

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e01-final-dashboard-alert-ui.png" alt="Final dashboard SQL Injection alert">
  <figcaption>Hình 1 - Dashboard cuối hiển thị alert SQL Injection mức Critical đã được persist cùng AI2B, Fusion, MITRE và network context.</figcaption>
</figure>

**Quan sát:** Ảnh này xác thực trạng thái alert cuối được hiển thị. Ảnh không được dùng để claim worker broadcast alert sang dashboard qua realtime WebSocket.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e02-s3-data-bucket-deleted.png" alt="S3 data bucket object cleanup evidence">
  <figcaption>Hình 2 - Object trong S3 data bucket đã được xóa trước bước xóa bucket sau khi xử lý evidence và artifact.</figcaption>
</figure>

**Quan sát:** Data/artifact bucket chỉ được cleanup sau khi evidence, model artifacts, receipts và screenshots đã được archive bên ngoài bucket.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e04-rds-final-snapshot-confirmation.png" alt="RDS final snapshot confirmation evidence">
  <figcaption>Hình 3 - Dialog delete RDS giữ tùy chọn tạo final snapshot trước khi xóa PostgreSQL instance.</figcaption>
</figure>

**Quan sát:** RDS deletion nên giữ final snapshot trừ khi có approval riêng để skip. Điều này giúp bảo vệ dữ liệu final alert sau khi live database bị gỡ bỏ.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e05-sqs-queues-deleted.png" alt="SQS queues deleted evidence">
  <figcaption>Hình 4 - Danh sách SQS queue không còn queue sau khi main queue và DLQ được xóa.</figcaption>
</figure>

**Quan sát:** Queue deletion diễn ra sau khi queue state và DLQ evidence decision đã được xử lý. Purge không bắt buộc nếu queue được xóa luôn.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e06-cloudfront-deferred-pro-plan.png" alt="CloudFront deferred cleanup due to Pro plan evidence">
  <figcaption>Hình 5 - CloudFront distributions vẫn bị giữ ở Pro / flat-rate pricing plan nên việc xóa hoàn toàn được deferred.</figcaption>
</figure>

**Quan sát:** Đây là cleanup limitation quan trọng nhất. Distribution, Web ACL và OAC cần ở deferred register cho tới khi AWS cho phép WAF disassociation và distribution deletion.

<figure class="worklog-evidence">
  <img src="/images/worklog/week12/w12-e07-s3-static-bucket-deleted.png" alt="S3 static bucket deleted evidence">
  <figcaption>Hình 6 - Frontend/static S3 bucket deletion được xác nhận sau khi không còn cần AWS public hosting cho demo.</figcaption>
</figure>

**Quan sát:** Cleanup frontend static bucket được tách khỏi report publication. Final FCAJ report được phục vụ từ Hugo/Vercel thay vì AWS S3 frontend bucket đã xóa.

**Ghi chú evidence:** Screenshot RDS action menu và ảnh Vercel live-site riêng không còn được lưu thành file public trước cleanup. Dialog RDS final snapshot bên trên ghi lại safeguard khi xóa; public URL cuối được xác thực qua Vercel deployment đã cấu hình. Trang này không trình bày screenshot bị thiếu như evidence.

## Ghi chú CloudFront deferred cleanup

CloudFront distribution cleanup was partial/deferred. The distribution remained under a CloudFront Pro / flat-rate pricing plan and was changing back to pay-as-you-go. WAF disassociation, distribution deletion, and OAC deletion were deferred until AWS allows the pricing-plan transition to complete.

Follow-up đúng là:

```text
1. Confirm CloudFront đã chuyển về pay-as-you-go.
2. Giữ hoặc set distribution disabled.
3. Đợi CloudFront status là Deployed.
4. Disassociate CloudFront-created Web ACL.
5. Delete unused Web ACL.
6. Delete distribution bằng latest ETag.
7. Delete OAC chỉ khi không còn distribution nào reference nó.
```

Kết quả cleanup Tuần 12 vì vậy được ghi như sau:

| Resource | Kết quả cleanup | Lý do |
| --- | --- | --- |
| CloudFront distribution | Deferred | Pro / flat-rate pricing-plan transition |
| CloudFront-created WAF Web ACL | Deferred | Required while pricing plan association remains |
| CloudFront OAC | Deferred | Distribution vẫn reference OAC |
| S3, SQS, RDS, Secrets | Đã xử lý theo evidence và approval gates | Cleanup actions đã completed hoặc scheduled khi AWS hỗ trợ |

## Hugo và Vercel publication

Sau AWS cleanup, workshop report cần một public review URL ổn định. Hugo được dùng để build static report, và Vercel được dùng làm public hosting layer cho FCAJ review.

Checklist publication:

```text
[ ] Hugo local build passes.
[ ] Worklog Week 11 và Week 12 render được.
[ ] Cleanup page render được ở EN và VI.
[ ] Image paths trỏ tới curated public-safe screenshots.
[ ] Không publish raw evidence folders.
[ ] Vercel production URL mở đúng.
[ ] Language switch vẫn ở Vercel domain.
```

## Kết quả cuối và bài học

Tuần 12 đóng dự án bằng evidence-first cleanup và public report delivery. SOC AI platform có đủ evidence để thể hiện final dashboard/API result, SQS/RDS persistence, idempotency behavior và cleanup discipline.

Bài học quan trọng nhất là cleanup cũng là một phần của engineering. Một số resource có thể xóa ngay, một số cần final snapshot hoặc recovery window, và một số phải deferred vì AWS service rules hoặc pricing-plan transition chặn thao tác xóa. Ghi nhận ranh giới đó trung thực làm final report chắc hơn việc giả vờ mọi resource đều đã được xóa cùng lúc.
