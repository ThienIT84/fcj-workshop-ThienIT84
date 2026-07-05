---
title: "Blog 2"
date: 2026-06-09
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# CLICKFORCE TĂNG TỐC PHÂN TÍCH QUẢNG CÁO VỚI AMAZON BEDROCK AGENTS

Trong bối cảnh Generative AI và AI Agent ngày càng phát triển, mình thấy một trong những bài toán thực tế nhất mà doanh nghiệp đang gặp phải chính là: phân tích dữ liệu quảng cáo thủ công quá chậm, không theo kịp tốc độ ra quyết định.

![Kiến trúc hệ thống Lumos của CLICKFORCE](/images/3-BlogsTranslated/blog2.png)

CLICKFORCE — công ty quảng cáo kỹ thuật số hàng đầu Đài Loan — từng cần đến 2–6 tuần chỉ để tạo một báo cáo phân tích ngành, vì phải thu thập dữ liệu, xử lý, phân tích và tổng hợp insight qua nhiều bước thủ công.

Để giải quyết vấn đề này, họ đã xây dựng hệ thống Lumos bằng cách kết hợp Amazon Bedrock Agents, Amazon Bedrock Knowledge Bases, Amazon SageMaker AI, Amazon OpenSearch Service, AWS Glue, AWS Lambda và Amazon ECS.

Giải pháp giúp tự động hóa toàn bộ quy trình phân tích, cho phép truy vấn dữ liệu bằng ngôn ngữ tự nhiên và tạo báo cáo thông minh trong thời gian ngắn hơn rất nhiều.

---

## MỘT SỐ ĐIỂM NỔI BẬT

- **Xây dựng AI Agent phân tích dữ liệu:**
  Amazon Bedrock Agents giúp hệ thống hiểu yêu cầu của người dùng, lập kế hoạch xử lý và gọi các công cụ phù hợp để phân tích dữ liệu.

- **Giảm hallucination bằng Knowledge Bases:**
  Amazon Bedrock Knowledge Bases giúp hệ thống truy xuất thông tin từ dữ liệu thực tế như báo cáo ngành, tài liệu chiến dịch và dữ liệu quảng cáo — thay vì để LLM tự "bịa" câu trả lời.

- **Tự động truy vấn dữ liệu bằng Text-to-SQL:**
  Người dùng đặt câu hỏi bằng tiếng người, hệ thống tự chuyển thành truy vấn SQL để lấy đúng dữ liệu cần thiết. Không cần biết SQL vẫn dùng được.

- **Tối ưu độ chính xác bằng Amazon SageMaker AI:**
  SageMaker AI được dùng để theo dõi, đánh giá và cải thiện pipeline Text-to-SQL liên tục, giúp kết quả truy vấn chính xác và ổn định hơn theo thời gian.

- **Xử lý và truy xuất dữ liệu hiệu quả hơn:**
  AWS Glue lo xử lý và chuẩn hóa dữ liệu, còn Amazon OpenSearch Service lo lập chỉ mục và tìm kiếm nhanh khi cần truy xuất.

- **Tối ưu chi phí và thời gian vận hành:**
  Sau khi triển khai, CLICKFORCE giảm thời gian tạo báo cáo từ 2–6 tuần xuống còn dưới 1 giờ, đồng thời giảm 47% chi phí vận hành.

---

## CÁCH TIẾP CẬN PHÂN TÍCH DỮ LIỆU THẾ HỆ MỚI

Thay vì phụ thuộc hoàn toàn vào đội data analyst hoặc phải viết truy vấn thủ công, người dùng có thể đặt câu hỏi trực tiếp bằng ngôn ngữ tự nhiên.

Amazon Bedrock Agents sẽ hiểu yêu cầu, xác định nguồn dữ liệu phù hợp, gọi đúng công cụ và trả về kết quả phân tích.

Ví dụ, đội marketing có thể hỏi:

"Chiến dịch nào có hiệu quả tốt nhất trong tháng này?"

"Nhóm khách hàng nào phản hồi tích cực nhất với quảng cáo mới?"

Hệ thống tự truy xuất dữ liệu, phân tích kết quả và hỗ trợ tạo insight để ra quyết định nhanh hơn.

---

## KẾT LUẬN

Điều mình rút ra từ case study này: Amazon Bedrock Agents không chỉ phù hợp để xây chatbot, mà còn có thể trở thành một AI Agent hỗ trợ phân tích dữ liệu doanh nghiệp thực sự.

Khi kết hợp với Knowledge Bases, SageMaker AI, OpenSearch, Glue và Lambda, doanh nghiệp có thể rút ngắn thời gian phân tích, giảm chi phí vận hành và khai thác dữ liệu hiệu quả hơn.
