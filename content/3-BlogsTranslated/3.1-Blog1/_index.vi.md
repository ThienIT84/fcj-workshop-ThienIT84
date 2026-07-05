---
title: "Blog 1"
date: 2026-06-04
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# TỰ ĐỘNG HÓA QUẢN LÝ CHI PHÍ AWS VỚI FINOPS AGENT TRÊN AMAZON BEDROCK AGENTCORE

Khi doanh nghiệp sử dụng nhiều AWS account và nhiều dịch vụ cloud, việc kiểm soát chi phí không còn đơn giản là chỉ xem báo cáo cuối tháng. Để giải quyết vấn đề này, AWS giới thiệu FinOps Agent được xây dựng trên Amazon Bedrock AgentCore, giúp người dùng hỏi dữ liệu chi phí AWS bằng ngôn ngữ tự nhiên, nhận phân tích về cost driver, budget, anomaly và các đề xuất tối ưu tài nguyên. Theo mình, điểm hay của giải pháp này là Agent không thay thế các công cụ như Cost Explorer hay Compute Optimizer, mà đóng vai trò gom dữ liệu, diễn giải kết quả và hỗ trợ đưa ra hành động tối ưu chi phí nhanh hơn.

![Kiến trúc FinOps Agent trên Amazon Bedrock AgentCore](/images/3-BlogsTranslated/blog1.png)

---

## MỘT SỐ ĐIỂM NỔI BẬT

- **Sử dụng Amazon Bedrock AgentCore**: AgentCore được dùng để chạy FinOps Agent, xử lý câu hỏi, gọi đúng công cụ AWS cần thiết và trả về kết quả phân tích chi phí bằng ngôn ngữ dễ hiểu.
- **Khai thác AWS Cost Explorer**: Agent có thể xác định service, account, region hoặc usage type nào đang đóng góp nhiều nhất vào chi phí. Ví dụ, khi chi phí tăng bất thường, Agent có thể phân tích nguyên nhân đến từ EC2, S3, NAT Gateway, RDS hoặc data transfer.
- **Khai thác AWS Budgets**: Agent có thể so sánh mức sử dụng hiện tại với ngân sách, cảnh báo nguy cơ vượt budget và gợi ý nhóm cần kiểm tra.
- **Khai thác AWS Compute Optimizer**: Agent có thể lấy recommendation để phát hiện tài nguyên đang over-provisioned, ví dụ EC2 instance cấu hình quá cao so với nhu cầu thực tế, từ đó gợi ý resize instance hoặc điều chỉnh workload phù hợp hơn.
- **Hỏi chi phí bằng ngôn ngữ tự nhiên**: Người dùng có thể hỏi: "Dịch vụ nào đang tốn nhiều chi phí nhất tháng này?", "Account nào có mức tăng chi phí bất thường?" hoặc "Có tài nguyên EC2 nào đang dùng chưa hiệu quả không?".
- **Hỗ trợ phối hợp giữa Finance và DevOps**: Finance có thể hỏi về ngân sách, còn DevOps có thể đi sâu vào tài nguyên, workload và recommendation kỹ thuật. Điều này giúp các nhóm cùng hiểu vấn đề chi phí theo một cách dễ tiếp cận hơn.

---

## GÓC NHÌN CÁ NHÂN

Theo mình, FinOps Agent có giá trị thực tế nhất khi doanh nghiệp đã dùng nhiều AWS service và chi phí cloud không còn dễ kiểm soát thủ công. Khi đó, vấn đề không chỉ là "chi phí bao nhiêu", mà là "vì sao tăng", "tăng ở đâu", "ai chịu trách nhiệm" và "có thể tối ưu bằng hành động nào".

Ví dụ, nếu Cost Explorer cho thấy chi phí EC2 tăng, con số đó chưa đủ để ra quyết định. Doanh nghiệp cần biết instance nào tăng, có chạy đúng workload không, có bị bỏ quên không, có thể rightsizing không, hay nên cân nhắc Savings Plans. Đây là lúc FinOps Agent có thể gom nhiều tín hiệu lại để đưa ra phân tích dễ hiểu hơn.

---

## MỘT SỐ HẠN CHẾ CẦN LƯU Ý

- AI Agent không thay thế hoàn toàn chuyên gia FinOps, đặc biệt với production workload.
- Chất lượng phân tích phụ thuộc nhiều vào dữ liệu đầu vào như tagging, account structure và cost allocation.
- Rightsizing không nên áp dụng máy móc, vì một số workload vẫn cần headroom cho giờ cao điểm.
- Cần kiểm soát quyền truy cập vì Agent có thể truy vấn dữ liệu chi phí nhạy cảm.
- Bản thân Agent cũng phát sinh chi phí inference, nên vẫn cần theo dõi mức sử dụng.

---

## KẾT LUẬN

Amazon Bedrock AgentCore không chỉ dùng để xây chatbot, mà có thể ứng dụng trực tiếp vào bài toán quản lý và tối ưu chi phí cloud. Khi kết hợp với Cost Explorer, Budgets, Compute Optimizer và Pricing, FinOps Agent giúp doanh nghiệp kiểm soát chi phí chủ động hơn, phát hiện lãng phí sớm hơn và ra quyết định tối ưu tài nguyên nhanh hơn. Tuy nhiên, để giải pháp hiệu quả, doanh nghiệp vẫn cần nền tảng dữ liệu tốt như tagging rõ ràng, phân tách account hợp lý, budget được cấu hình đúng và có quy trình review recommendation trước khi áp dụng.
