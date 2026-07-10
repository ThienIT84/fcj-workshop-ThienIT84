---
title: "Chia sẻ, đóng góp ý kiến"
date: 2026-07-10
weight: 7
chapter: false
pre: " <b> 7. </b> "
---
## Tổng kết trải nghiệm

Sau khi hoàn thành chương trình First Cloud Journey và các workshop, mình hiểu rõ hơn cách kiến thức cloud được áp dụng vào một dự án thực tế. Trước đó, mình thường tiếp cận các dịch vụ AWS như những thành phần riêng lẻ. Qua kỳ thực tập, mình đã thấy cách chúng kết hợp thành một hệ thống hoàn chỉnh.

Phần có giá trị nhất là xây dựng và triển khai **AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform**. Dự án kết nối hạ tầng cloud, backend, mô hình AI, giám sát bảo mật và quy trình vận hành/cleanup trong một luồng làm việc thống nhất.

## Môi trường học tập và làm việc

First Cloud Journey tạo ra một môi trường học tập thực hành và cởi mở. Cấu trúc workshop khuyến khích tự triển khai, vì vậy mình hiểu sâu hơn các khái niệm AWS so với chỉ đọc tài liệu hoặc xem hướng dẫn.

Trong quá trình thực hiện, mình phải kiểm tra AWS Console, đọc lỗi, rà soát dependency giữa các dịch vụ và so sánh kiến trúc dự kiến với trạng thái đã deploy. Các tình huống như WAF chặn request XSS, quyền IAM cho worker, SQS idempotency, RDS final snapshot và CloudFront pricing-plan transition đã giúp trải nghiệm sát với công việc cloud engineering thực tế.

## Sự hỗ trợ từ mentor và team

Mentor và team hỗ trợ theo hướng gợi mở cách phân tích vấn đề thay vì chỉ đưa đáp án. Mình học được cách đặt câu hỏi kỹ thuật rõ ràng hơn, thu thập evidence, kiểm tra hành vi của từng dịch vụ AWS và giải thích quyết định của mình.

Làm việc theo nhóm cũng giúp mình hiểu tầm quan trọng của việc phân chia trách nhiệm, ghi nhận quyết định và xác nhận dependency trước khi xóa tài nguyên dùng chung.

## Giá trị chuyên môn

Dự án kết hợp các phần mình quan tâm: machine learning, cloud engineering và cybersecurity. Mình làm việc với Zeek logs, traffic HTTP, adapter AI, fusion logic, API backend, SQS/DLQ, RDS PostgreSQL và dashboard cho analyst.

Mình cũng hiểu rằng một mô hình AI không tự tạo thành sản phẩm có thể vận hành. Model cần được tích hợp với API, queue, database, storage, monitoring, access control và giao diện người dùng. Điều đó giúp mình nhìn rõ hơn toàn bộ vòng đời từ model artifact đến hệ thống deploy.

## Kỹ năng và bài học chính

- Thiết kế dependency giữa VPC, EC2, ALB, CloudFront, S3, SQS, RDS và CloudWatch.
- Phân tách human access qua IAM Identity Center với runtime access qua IAM roles.
- Lưu credential RDS trong Secrets Manager và kiểm tra quyền tối thiểu cho backend/worker.
- Kiểm thử SQL Injection và XSS theo luồng API, worker, RDS và dashboard.
- Xử lý idempotency cho SQS Standard bằng `event_id` và upsert ở bảng `final_alerts`.
- Thực hiện cleanup theo nguyên tắc evidence-first, final snapshot và recovery window.

Một bài học quan trọng là cloud engineering không kết thúc khi ứng dụng chạy được. Monitoring, kiểm soát chi phí, quản lý quyền, lưu evidence và cleanup đúng dependency cũng là một phần của hệ thống.

## Đề xuất cho chương trình

Chương trình đã rất hữu ích. Một số nội dung có thể giúp các khóa sau làm workshop hiệu quả hơn:

- Có checklist cleanup chuẩn cho các tài nguyên AWS phổ biến.
- Có hướng dẫn ngắn về kiểm soát chi phí sau lab/workshop.
- So sánh rõ IAM Identity Center, IAM users, IAM roles và permission sets.
- Bổ sung ví dụ về CloudFront, WAF và pricing-plan behavior.
- Cung cấp evidence register để học viên biết ảnh nào cần chụp và trường nào cần mask trước khi public.

## Định hướng tiếp theo

Sau kỳ thực tập, mình muốn tiếp tục phát triển kỹ năng về AWS architecture, cloud security, DevOps, MLOps và giám sát an ninh có hỗ trợ AI. Với SOC AI platform, các hướng cải tiến tiếp theo gồm đánh giá model kỹ hơn, cải thiện dashboard, tự động hóa deployment, infrastructure as code và observability.

Tổng thể, First Cloud Journey giúp mình chuyển từ việc học các dịch vụ cloud riêng lẻ sang hiểu cách xây dựng, vận hành, bảo vệ và cleanup một hệ thống cloud thực tế.
