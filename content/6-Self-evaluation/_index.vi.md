---
title: "Tự đánh giá"
date: 2026-07-12
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

## Bối cảnh thực tập

Quá trình thực tập của tôi được thực hiện tại **Công ty TNHH Amazon Web Services Việt Nam** trong thời gian từ **20/04/2026 đến 12/07/2026**. Trong giai đoạn này, tôi thực hiện đề tài **AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform**, một nền tảng giám sát an ninh kết hợp hạ tầng cloud, dữ liệu telemetry mạng, kết quả từ nhiều mô hình AI, backend xử lý alert và dashboard cho người phân tích.

Dự án yêu cầu tôi kết nối nhiều mảng kiến thức thường được học riêng lẻ: AWS networking, IAM, storage, database, log bảo mật dạng Zeek/Suricata, đánh giá mô hình AI, tích hợp backend FastAPI, xử lý bất đồng bộ bằng SQS, lưu trữ alert bằng RDS, lưu evidence triển khai và cleanup tài nguyên sau demo. Nhờ đó, tôi hiểu rõ hơn sự khác nhau giữa một chức năng chạy được ở local và một hệ thống có thể deploy, kiểm thử, ghi tài liệu và dọn dẹp có trách nhiệm.

## Đóng góp chính

Trong 12 tuần thực tập, các phần đóng góp chính của tôi gồm:

| Mảng công việc | Đóng góp |
| --- | --- |
| Nền tảng AWS | Thực hành IAM, VPC, subnet, route table, EC2, Security Group, S3, CloudFront, RDS, CloudWatch và kiểm soát chi phí |
| Dữ liệu và telemetry | Xây dựng, rà soát nguồn traffic lab bằng Zeek log, HTTP event, connection log và các profile normal/attack |
| Mô hình AI | Hỗ trợ luồng AI2A và AI2B, gồm rà soát feature, kiểm tra chất lượng dữ liệu, ghi chú freeze model và kiểm thử dự đoán SQL Injection/XSS |
| Tích hợp backend | Kết nối FastAPI ingestion, model adapter, evidence routing, fusion decision output và API contract cho alert |
| Triển khai AWS | Kiểm thử thin-slice deployment với CloudFront, ALB, EC2 backend, API route, SQS, DLQ, worker service và RDS PostgreSQL |
| Persistence và độ tin cậy | Xác thực RDS `final_alerts`, `/api/alerts/latest` và kiểm thử D5-5 idempotency theo `event_id` |
| Báo cáo cuối | Tổ chức worklog evidence, tài liệu cleanup, hướng dẫn deploy Vercel và nội dung workshop để nộp cuối kỳ |

## Bảng tự đánh giá

| STT | Tiêu chí | Mức tự đánh giá | Minh chứng từ quá trình thực tập |
| --- | --- | --- | --- |
| 1 | Kiến thức và kỹ năng chuyên môn | Tốt | Tôi đã áp dụng kiến thức AWS, backend, security monitoring, AI model và database trong một dự án SOC AI tích hợp. |
| 2 | Khả năng học hỏi | Tốt | Tôi học và xử lý nhiều phần mới như deployment, IAM, SQS/RDS, worker, dashboard và cleanup AWS. |
| 3 | Tính chủ động | Tốt | Tôi chủ động kiểm thử API, xem log, truy nguyên lỗi dashboard và chuẩn bị runbook cleanup/deploy thay vì chỉ chờ hướng dẫn cố định. |
| 4 | Tinh thần trách nhiệm | Tốt | Tôi lưu evidence, tránh claim quá mức, ghi nhận final snapshot cho RDS và nêu rõ CloudFront/WAF/OAC là cleanup deferred. |
| 5 | Kỷ luật và tính nhất quán | Khá đến Tốt | Tôi duy trì worklog 12 tuần và hoàn thiện báo cáo cuối, nhưng cần tổ chức ảnh/evidence sớm hơn để tránh dồn việc gần deadline. |
| 6 | Giao tiếp và báo cáo | Khá đến Tốt | Tôi viết được tài liệu kỹ thuật và runbook, nhưng cần trình bày trạng thái ngắn gọn, rõ nguyên nhân và hành động hơn khi báo cáo với người khác. |
| 7 | Hợp tác nhóm | Tốt | Tôi bám theo cấu trúc workshop FCAJ, tiếp nhận định hướng từ mentor và chuyển tiến độ kỹ thuật thành nội dung có thể review. |
| 8 | Tư duy giải quyết vấn đề | Tốt | Tôi đã debug các lỗi liên quan đến frontend API, giả định WebSocket, CloudFront/WAF, quyền S3, SQS/RDS wiring và cleanup dependency. |
| 9 | Nhận thức về bảo mật và chi phí | Tốt | Tôi chú ý đến least privilege, lưu evidence, S3 versioning, RDS snapshot, Secrets Manager recovery window, DLQ và thứ tự cleanup AWS. |
| 10 | Đóng góp cho dự án | Tốt | Tôi góp phần đưa dự án từ thử nghiệm local AI/backend thành luồng có API deploy, alert persist, idempotency evidence, cleanup docs và public report site. |
| 11 | Đánh giá tổng thể | Tốt | Tôi hoàn thành kỳ thực tập với một câu chuyện kỹ thuật rõ ràng, có evidence, có giới hạn được ghi nhận và có đường dẫn nộp cuối. |

## Điểm mạnh

Điểm tiến bộ rõ nhất của tôi trong kỳ thực tập là khả năng nối nhiều lớp kỹ thuật thành một luồng hoạt động thống nhất. Ở giai đoạn đầu, tôi nhìn cloud service, AI model, backend API và security log như các phần khá rời rạc. Đến cuối kỳ, tôi có thể phân tích một luồng end-to-end như:

```text
Zeek / replay event
-> CloudFront / ALB
-> FastAPI backend
-> SQS main queue
-> worker processing
-> RDS final_alerts
-> API / dashboard evidence
```

Tôi cũng cải thiện tư duy debug. Thay vì chỉ kiểm tra một command có chạy thành công hay không, tôi học cách đối chiếu log, API response, frontend rendering, dữ liệu trong database, trạng thái queue và evidence trên AWS console. Điều này đặc biệt quan trọng khi backend API đã có dữ liệu nhưng dashboard chưa render alert mới, hoặc khi CloudFront/WAF khiến một số bước cleanup phải deferred.

Một điểm mạnh khác là cách giữ evidence. Tôi học được rằng báo cáo cuối sẽ đáng tin hơn nếu ghi rõ phần nào đã chứng minh, phần nào chưa chứng minh và phần nào bị deferred. Ví dụ, tôi không claim worker-to-dashboard realtime WebSocket broadcast khi chưa có evidence riêng, và ghi CloudFront/WAF/OAC là cleanup deferred thay vì nói đã xóa hoàn toàn.

## Điểm cần cải thiện

Điểm đầu tiên tôi cần cải thiện là tổ chức sớm hơn. Một số screenshot, file evidence và đường dẫn tài liệu được xử lý khá muộn. Ở các dự án sau, tôi nên thống nhất tên thư mục evidence, quy tắc đặt tên ảnh và cấu trúc báo cáo ngay từ đầu để giai đoạn publish cuối nhẹ hơn.

Điểm thứ hai là giao tiếp. Tôi có thể giải thích chi tiết kỹ thuật, nhưng đôi khi đưa quá nhiều thông tin trung gian cùng lúc. Tôi cần luyện cách báo cáo ngắn hơn theo cấu trúc: trạng thái hiện tại, nguyên nhân chính, hành động tiếp theo và rủi ro còn lại.

Điểm thứ ba là lập kế hoạch tích hợp frontend sớm hơn. Evidence backend/API/RDS khá chắc, nhưng phần dashboard rendering và realtime behavior cần validation riêng. Trong các dự án sau, tôi nên xác định test cho UI từ sớm, gồm API polling, WebSocket behavior, stale build/cache và biến môi trường production.

## Tổng kết cá nhân

Kỳ thực tập này giúp tôi nhận ra rằng xây dựng một nền tảng giám sát an ninh không chỉ là huấn luyện mô hình hoặc deploy một server. Một hệ thống có ích còn cần dữ liệu sạch, API ổn định, IAM permission đúng, log quan sát được, storage có persistence, frontend tích hợp cẩn thận và quy trình cleanup bảo vệ cả chi phí lẫn evidence.

Bài học quan trọng nhất với tôi là phải trung thực với ranh giới của hệ thống. Một dự án vẫn có thể mạnh nếu nói rõ phần nào đã hoàn thành, phần nào có evidence chứng minh và phần nào cần follow-up. Tư duy đó giúp tôi hoàn thiện SOC AI workshop theo hướng thực tế hơn thay vì chỉ tạo một demo bề mặt.
