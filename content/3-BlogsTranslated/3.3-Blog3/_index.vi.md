---
title: "Blog 3"
date: 2026-06-24
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# AI NỘI BỘ CHO DOANH NGHIỆP: KHI NHÂN VIÊN KHÔNG CẦN LỤC TỪNG FOLDER ĐỂ TÌM THÔNG TIN

![Kiến trúc AI Assistant nội bộ với Amazon Bedrock](/images/3-BlogsTranslated/blog3.jpg)

Dữ liệu trong doanh nghiệp thường không nằm ở một nơi duy nhất. Một phần có thể nằm trong Google Drive. Một phần nằm trong SharePoint. Một phần nằm trong CRM, ERP. Quy trình nội bộ có thể nằm trong file PDF. FAQ khách hàng có thể nằm trong ticket system. Một số thông tin khác lại nằm trong email, file Excel hoặc folder riêng của từng phòng ban.

Vấn đề không phải là doanh nghiệp không có dữ liệu. Vấn đề là dữ liệu bị phân tán, khó tìm, khó kiểm soát phiên bản và đôi khi nhân viên không biết đâu mới là tài liệu mới nhất. Một bạn sales muốn tìm chính sách chiết khấu mới nhất. Một bạn HR muốn tra lại quy trình nghỉ phép. Một bạn support muốn biết cách xử lý một lỗi khách hàng thường gặp. Một nhân viên mới muốn xem tài liệu onboarding.

Nếu phải tìm thủ công qua nhiều hệ thống khác nhau, việc này sẽ mất thời gian. Nghiêm trọng hơn, nhân viên có thể dùng nhầm tài liệu cũ hoặc thông tin không còn hiệu lực. Từ vấn đề đó, một câu hỏi rất thực tế được đặt ra: Có thể xây một AI Assistant nội bộ để nhân viên hỏi đáp trực tiếp trên tài liệu công ty không?

Không cần bắt đầu bằng một hệ thống quá lớn. Không cần làm một AI biết toàn bộ mọi thứ trong doanh nghiệp ngay từ ngày đầu tiên. Phiên bản đầu tiên chỉ cần đủ đơn giản để kiểm tra xem ý tưởng này có thật sự hữu ích hay không.

---

## 1. BẮT ĐẦU TỪ MỘT PHẠM VI NHỎ

Nếu triển khai AI Assistant cho toàn bộ doanh nghiệp ngay từ đầu, bài toán sẽ rất phức tạp. Mỗi phòng ban có kiểu dữ liệu khác nhau. Mỗi hệ thống có cách lưu trữ khác nhau. Mức độ nhạy cảm của tài liệu cũng khác nhau. Nếu làm quá rộng ngay từ đầu, khi hệ thống trả lời chưa tốt, team sẽ rất khó biết vấn đề nằm ở đâu.

Cách hợp lý hơn là chọn một phạm vi nhỏ trước. Ví dụ có thể bắt đầu với: Chính sách nhân sự, quy trình nghỉ phép, tài liệu onboarding cho nhân viên mới, FAQ sản phẩm, quy trình xử lý ticket phổ biến, tài liệu hướng dẫn nội bộ thường được hỏi lại nhiều lần.

Những nhóm tài liệu này có điểm chung là nội dung tương đối rõ ràng, dễ kiểm chứng và thường xuyên được nhân viên hỏi lại. Mục tiêu ban đầu không phải là làm một AI biết tất cả mọi thứ trong công ty. Mục tiêu thực tế hơn là: Nhân viên đặt câu hỏi, hệ thống tìm trong tài liệu nội bộ đã được xác nhận, sau đó trả lời kèm nguồn tài liệu rõ ràng.

Ví dụ người dùng hỏi: “Em muốn xin nghỉ phép 2 ngày thì cần làm gì?” Một câu trả lời bình thường có thể là: “Bạn cần làm đơn xin nghỉ phép và gửi quản lý phê duyệt.” Nghe có vẻ đúng, nhưng trong môi trường doanh nghiệp thì chưa đủ. Một câu trả lời tốt hơn nên là: “Theo tài liệu Quy trình nghỉ phép năm 2026, nhân viên cần tạo đơn nghỉ phép trên hệ thống HRM trước ít nhất 3 ngày làm việc và chờ quản lý trực tiếp phê duyệt. Nguồn: Quy trình nghỉ phép năm 2026 - HR Team.”

Điểm khác biệt nằm ở phần nguồn. Với dữ liệu doanh nghiệp, câu trả lời nghe hợp lý thôi là chưa đủ. Người dùng cần biết câu trả lời đó đến từ tài liệu nào, còn hiệu lực hay không và có thể kiểm chứng lại được không.

---

## 2. KIẾN TRÚC AWS Ở PHIÊN BẢN ĐẦU TIÊN

Ở phiên bản đầu tiên, kiến trúc có thể giữ đơn giản:
Tài liệu nội bộ → Amazon S3 → Amazon Bedrock Knowledge Bases → AWS Lambda + API Gateway → Web chat nội bộ → Nhân viên nhận câu trả lời kèm nguồn.

Cách hiểu đơn giản là doanh nghiệp chọn một bộ tài liệu đã được kiểm tra, sau đó lưu trữ trên Amazon S3. Amazon Bedrock Knowledge Bases hỗ trợ lập chỉ mục và truy xuất những nội dung liên quan khi người dùng đặt câu hỏi. Thay vì để AI trả lời theo cảm tính, hệ thống sẽ dựa trên tài liệu nội bộ đã được cung cấp để tạo ra câu trả lời phù hợp hơn.

AWS Lambda đảm nhiệm phần xử lý backend, ví dụ như nhận câu hỏi từ người dùng, gọi đến Knowledge Base, nhận kết quả truy xuất và trả phản hồi về cho giao diện web. Amazon API Gateway đóng vai trò là cổng API giữa frontend và backend, giúp web chat nội bộ có thể giao tiếp với Lambda một cách rõ ràng và dễ quản lý.

Nếu cần đăng nhập, doanh nghiệp có thể bổ sung Amazon Cognito để chỉ nhân viên được phép truy cập hệ thống. Amazon CloudWatch có thể được dùng để theo dõi log, kiểm tra lỗi và ghi nhận những câu hỏi mà hệ thống chưa trả lời tốt.

Điểm quan trọng là phiên bản đầu tiên chưa cần kết nối toàn bộ hệ thống như CRM, ERP, Google Drive hay SharePoint. Doanh nghiệp có thể bắt đầu bằng cách chọn thủ công một nhóm tài liệu quan trọng, đưa lên Amazon S3 và thử nghiệm với một nhóm người dùng nhỏ. Cách làm này giúp team tập trung vào câu hỏi quan trọng nhất: AI Assistant này có thật sự giúp nhân viên tìm thông tin nhanh hơn, chính xác hơn và đáng tin cậy hơn không?

---

## 3. AMAZON BEDROCK MANAGED KNOWLEDGE BASE CÓ GÌ ĐÁNG CHÚ Ý?

Khi xây một hệ thống hỏi đáp trên dữ liệu nội bộ, nhiều team sẽ nghĩ đến RAG, tức Retrieval-Augmented Generation. Hiểu đơn giản, RAG là cách để AI không chỉ dựa vào kiến thức có sẵn của model, mà còn truy xuất thêm dữ liệu từ nguồn tài liệu riêng của doanh nghiệp trước khi tạo câu trả lời.

Tuy nhiên, nếu tự xây một RAG pipeline từ đầu, team thường phải xử lý khá nhiều phần: Kết nối dữ liệu từ nhiều nguồn, làm sạch và chuẩn hóa tài liệu, chia nhỏ nội dung tài liệu, tạo embeddings, quản lý vector database, tối ưu retrieval, re-ranking kết quả, theo dõi log và hiệu năng, kiểm soát quyền truy cập, cập nhật dữ liệu khi tài liệu thay đổi.

Những phần này không quá đơn giản, đặc biệt khi dữ liệu doanh nghiệp nằm ở nhiều nơi và có nhiều định dạng khác nhau. Amazon Bedrock Managed Knowledge Base giúp giảm bớt độ phức tạp này bằng cách quản lý nhiều thành phần của RAG pipeline, từ ingestion, indexing, storage cho đến retrieval infrastructure.

Điểm đáng chú ý là developer có thể tập trung nhiều hơn vào ứng dụng, trải nghiệm người dùng và logic nghiệp vụ, thay vì phải tự dựng toàn bộ hạ tầng truy xuất dữ liệu từ đầu. Một số điểm nổi bật có thể kể đến: Hỗ trợ kết nối dữ liệu từ các nguồn như Amazon S3, SharePoint, Confluence, Google Drive, OneDrive và Web Crawler; hỗ trợ xử lý nhiều dạng dữ liệu khác nhau để phục vụ việc truy xuất thông tin; có khả năng retrieval nâng cao cho các câu hỏi phức tạp; có thể trả lời kèm citations để người dùng kiểm tra nguồn; có thể tích hợp với Amazon Bedrock AgentCore để xây dựng AI Agent có khả năng truy xuất tri thức nội bộ.

Nói ngắn gọn, thay vì phải tự xây toàn bộ hệ thống RAG, doanh nghiệp có thể dùng Amazon Bedrock Knowledge Bases hoặc Managed Knowledge Base như một lớp nền tảng để đưa dữ liệu nội bộ vào ứng dụng Generative AI nhanh hơn và có kiểm soát hơn.

---

## 4. ĐIỀU QUAN TRỌNG KHÔNG CHỈ LÀ MODEL, MÀ LÀ DỮ LIỆU

Khi nói về AI, nhiều người thường hỏi: “Dùng model nào?”, “Model nào mạnh nhất?”, “Model nào trả lời hay nhất?”. Nhưng với bài toán hỏi đáp tài liệu nội bộ, câu hỏi quan trọng hơn nên là: “Dữ liệu có sạch không?”, “Tài liệu còn hiệu lực không?”, “Nguồn có rõ ràng không?”, “Ai chịu trách nhiệm cập nhật tài liệu?”, “Có nhiều phiên bản trùng nhau không?”, “Người dùng có được quyền xem tài liệu đó không?”.

Nếu dữ liệu bị cũ, trùng lặp hoặc không rõ nguồn, AI rất dễ trả lời sai. Nếu cùng một chính sách có ba phiên bản khác nhau, hệ thống cũng khó biết nên dựa vào bản nào. Vì vậy, trước khi đưa tài liệu vào AI Assistant, doanh nghiệp nên có một bước chọn lọc và chuẩn hóa dữ liệu.

Không cần làm quá phức tạp ngay từ đầu, nhưng mỗi tài liệu nên có một số thông tin cơ bản như: Tên tài liệu, phòng ban phụ trách, loại tài liệu, ngày hiệu lực, Owner, link nguồn, trạng thái còn hiệu lực hoặc đã lỗi thời.

Ví dụ:
- Tên tài liệu: Quy trình nghỉ phép năm 2026
- Phòng ban: HR
- Loại tài liệu: Policy
- Ngày hiệu lực: 01/01/2026
- Owner: HR Team

Những thông tin nhỏ này giúp hệ thống trả lời đáng tin cậy hơn. Đồng thời, team vận hành cũng dễ biết tài liệu nào cần cập nhật, tài liệu nào cần loại bỏ và tài liệu nào đang được người dùng hỏi nhiều nhất.

---

## 5. NHỮNG GÌ NÊN LÀM Ở BẢN ĐẦU TIÊN

Theo mình, phiên bản đầu tiên nên tập trung vào những việc thật sự cần thiết.

**Nên làm:**
- Chọn một phòng ban hoặc một nhóm tài liệu cụ thể
- Chỉ dùng tài liệu đã được xác nhận còn hiệu lực
- Làm một giao diện hỏi đáp đơn giản
- Câu trả lời phải có nguồn
- Ghi lại những câu hỏi hệ thống chưa trả lời tốt
- Cho người dùng đánh giá câu trả lời đúng hay chưa
- Kiểm tra xem AI Assistant có thật sự giúp tiết kiệm thời gian tìm kiếm không

**Chưa nên làm quá nhiều thứ cùng lúc:**
- Chưa cần kết nối toàn bộ hệ thống trong công ty
- Chưa cần dashboard quá phức tạp
- Chưa cần phân quyền quá chi tiết ngay từ đầu
- Chưa cần tích hợp Teams hoặc Slack ngay
- Chưa nên đưa tài liệu quá nhạy cảm vào giai đoạn thử nghiệm

Lý do là nếu phạm vi quá rộng, khi hệ thống trả lời chưa tốt, team sẽ rất khó xác định nguyên nhân. Có thể lỗi nằm ở dữ liệu chưa sạch, có thể do tài liệu bị chia nhỏ chưa hợp lý, có thể do prompt chưa tốt, có thể do quyền truy cập chưa rõ, cũng có thể do người dùng đang hỏi ngoài phạm vi tài liệu đã cung cấp. Bắt đầu nhỏ giúp việc kiểm tra, đo lường và cải thiện dễ hơn nhiều.

---

## 6. ĐO HIỆU QUẢ NHƯ THẾ NÀO?

Một hệ thống AI nội bộ không nên được đánh giá bằng cảm giác “có vẻ hay”. Sau một thời gian thử nghiệm, nên nhìn vào những chỉ số cụ thể.

Ví dụ:
- Có bao nhiêu người dùng thử?
- Mỗi ngày có bao nhiêu câu hỏi?
- Bao nhiêu câu trả lời có nguồn rõ ràng?
- Bao nhiêu câu hỏi không tìm thấy tài liệu phù hợp?
- Người dùng đánh giá câu trả lời đúng hay sai?
- Chủ đề nào được hỏi nhiều nhất?
- Những tài liệu nào thường xuyên được truy xuất?
- Những câu hỏi nào hệ thống trả lời chưa tốt?

Nếu nhiều câu hỏi không có câu trả lời, có thể bộ tài liệu đưa vào chưa đủ. Nếu câu trả lời có nguồn nhưng vẫn chưa chính xác, có thể cần xem lại cách tài liệu được xử lý, chia nhỏ hoặc gắn metadata. Nếu người dùng không sử dụng nhiều, có thể vấn đề được chọn chưa đủ “đau”, giao diện chưa thuận tiện hoặc nhân viên chưa thấy rõ giá trị của công cụ. Những phản hồi này quan trọng hơn việc cố gắng làm kiến trúc thật lớn ngay từ đầu.

---

## 7. KHI NÀO NÊN MỞ RỘNG?

Sau khi phiên bản đầu tiên chạy ổn và có người dùng thật, doanh nghiệp có thể mở rộng từng bước. Đầu tiên là bổ sung thêm tài liệu trong cùng phòng ban. Sau đó mở rộng sang phòng ban khác. Tiếp theo có thể đồng bộ tự động từ SharePoint, Google Drive, Confluence hoặc hệ thống nội bộ.

Khi dữ liệu nhiều hơn, lúc đó mới cần nghĩ kỹ hơn về phân quyền, audit log, dashboard, guardrails và tích hợp vào các công cụ làm việc hằng ngày như Teams, Slack hoặc portal nội bộ. Nói cách khác, hệ thống không cần trở thành “Enterprise Knowledge Hub” ngay từ ngày đầu tiên. Nó có thể bắt đầu như một công cụ nhỏ giúp nhân viên tìm tài liệu nhanh hơn, rồi dần phát triển thành một lớp truy cập tri thức chung cho doanh nghiệp.

---

## KẾT LUẬN

Dữ liệu phân tán là vấn đề rất phổ biến trong doanh nghiệp. Khi chưa có AI, vấn đề này đã khiến nhân viên mất nhiều thời gian tìm kiếm thông tin. Khi bắt đầu ứng dụng AI, vấn đề này càng rõ hơn, vì AI chỉ có thể trả lời tốt khi dữ liệu đầu vào đủ rõ ràng, cập nhật và đáng tin cậy.

Với AWS, một hướng tiếp cận thực tế là bắt đầu bằng một AI Assistant nội bộ cho một nhóm tài liệu nhỏ: → Amazon S3 → Amazon Bedrock Knowledge Bases → Lambda + API Gateway → Web chat nội bộ → Câu trả lời kèm nguồn.

Điều quan trọng nhất không phải là dùng thật nhiều dịch vụ AWS. Điều quan trọng là chọn đúng phạm vi, chọn đúng dữ liệu và luôn yêu cầu câu trả lời có nguồn. Vì trong môi trường doanh nghiệp, AI không chỉ cần trả lời nhanh. AI cần trả lời dựa trên đúng tài liệu, đúng ngữ cảnh và đủ tin cậy để người dùng có thể sử dụng trong công việc hằng ngày.
