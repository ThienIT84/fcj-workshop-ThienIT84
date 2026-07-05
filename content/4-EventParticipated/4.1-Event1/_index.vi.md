---
title: "FCAJ Community Day"
date: 2026-05-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch “FCAJ Community Day”

### Mục Đích Của Sự Kiện

* Học hỏi kinh nghiệm từ các anh chị trong cộng đồng FCAJ.
* Mở rộng góc nhìn về học tập, AI, cloud, mindset làm việc và phương pháp phát triển phần mềm.
* Kết nối với cộng đồng kỹ thuật.
* Rút ra định hướng học tập và phát triển bản thân trong quá trình internship.

### Danh Sách Diễn Giả

* **Anh Long** - Admin of FCAJ - *Tại sao não bạn chọn mạng xã hội thay vì việc học?*
* **Anh Thịnh** - DevOps/Cloud Engineer, FCAJ - *Automated Prompt Engineering: Nâng cao chất lượng đầu ra của LLM*
* **Anh Khang** - Solution Architect, Cloud Kinetics Vietnam - *AI-Ready Freshers: Kỹ năng và Mindset trong Kỷ nguyên AI*
* **Chị Thảo** - Software Developer, VIB - Ngân hàng Quốc tế Việt Nam - *Phương pháp BMAD: Đưa toàn bộ đội ngũ Agile & AI vào IDE của bạn*

### Nội Dung Nổi Bật

#### Anh Long - Tại sao não bạn chọn mạng xã hội thay vì việc học?

Phần chia sẻ tập trung vào lý do tại sao não bộ thường dễ dàng bị thu hút bởi mạng xã hội và game hơn là việc học. Nguyên nhân chính là vì những hoạt động này mang lại phần thưởng nhanh chóng, kích thích sự tò mò và ít tốn năng lượng hơn. Ngược lại, việc học thường có kết quả chậm và đòi hỏi sự tập trung lâu dài, dễ dẫn đến cảm giác chán nản hoặc trì hoãn.

Speaker đã giải thích cơ chế của dopamine và cách mạng xã hội hay game tạo ra cảm giác mong đợi phần thưởng liên tục. Để khắc phục điều này, người học có thể biến việc học thành một hoạt động hấp dẫn hơn bằng cách:
* Tạo ra các phần thưởng nhỏ sau khi hoàn thành mục tiêu.
* Duy trì chuỗi ngày học tập (streak, ví dụ như app Duolingo) để tạo động lực.
* Chia nhỏ mục tiêu học tập và bắt đầu bằng những tác vụ nhỏ, chẳng hạn như học trong 15 phút hoặc đọc 10 trang sách.

Bài học quan trọng nhất là: người học hiệu quả không chỉ dựa vào sự chăm chỉ nhất thời, mà cần biết cách xây dựng một môi trường và thói quen khiến bản thân muốn học mỗi ngày.

#### Anh Thịnh - Automated Prompt Engineering: Nâng cao chất lượng đầu ra của LLM

Đây là phần chia sẻ rất chi tiết về Prompt Engineering – nghệ thuật giao tiếp với AI để nhận được kết quả (output) chất lượng hơn. Nếu cung cấp một prompt chung chung, AI sẽ trả về kết quả chung chung; hướng dẫn mơ hồ sẽ tạo ra kết quả không nhất quán. Đặc biệt, những prompt kém chất lượng có thể gây lãng phí token, từ đó làm giảm năng suất và tăng chi phí.

Một prompt tốt và chuẩn chỉnh nên bao gồm các thành phần: **Role** (Vai trò), **Instruction/Task** (Nhiệm vụ), **Context** (Ngữ cảnh), **Input Data** (Dữ liệu đầu vào), **Output Format** (Định dạng đầu ra), **Examples** (Ví dụ) và **Constraints/Guidelines** (Ràng buộc/Hướng dẫn).

Diễn giả cũng chia sẻ những best practices quan trọng như:
* Sử dụng ngôn từ rõ ràng, cụ thể và directive language.
* Dùng các dấu phân cách (delimiter) để tách biệt các phần trong prompt.
* Mô tả cụ thể những điều AI *nên làm* thay vì *không nên làm*.
* Cho phép AI trả lời “I don’t know” nếu không có thông tin.
* Nên chia các input quá dài thành nhiều phần nhỏ để xử lý.

Về Token Economics, mô hình ngôn ngữ (LLM) xử lý nội dung theo từng token, chi phí phụ thuộc vào lượng input/output tokens và số lượng token tiêu tốn có thể khác nhau tùy vào ngôn ngữ sử dụng. Để tối ưu hơn, có thể áp dụng các kỹ thuật nâng cao (Advanced prompting techniques) như: Chain-of-Thought, Self-Consistency, Tree-of-Thoughts, Retrieval-Augmented Generation và Role Prompting.

Trong phiên chia sẻ, diễn giả đã giới thiệu dự án **Proptimizer** – một browser extension giúp tối ưu prompt và hỗ trợ chat với AI ngay trên trình duyệt. Solution architecture của Proptimizer sử dụng hoàn toàn các dịch vụ AWS bao gồm: CloudFront, S3, Cognito, API Gateway, Lambda, Bedrock, DynamoDB và CloudWatch. Kiến thức này mang lại giá trị thực tế cao, giúp sinh viên và intern biết cách sử dụng AI tốt hơn khi học AWS, viết tài liệu, debug lỗi hoặc xử lý các tác vụ kỹ thuật hàng ngày.

#### Anh Khang - AI-Ready Freshers: Kỹ năng và Mindset trong Kỷ nguyên AI

Phần trình bày nhấn mạnh rằng các fresher trong thời đại AI không nên chỉ chạy theo các công cụ mới một cách mù quáng, mà cần tập trung xây dựng kiến thức nền tảng (foundation) thật vững chắc. AI đóng vai trò như một bộ khuếch đại năng lực: nếu bạn có nền tảng tốt, AI sẽ giúp bạn làm tốt và nhanh hơn; nhưng nếu bạn không hiểu bản chất vấn đề, AI có thể khiến kết quả trở nên tệ hại và khó kiểm soát.

Khi đối mặt với các bài tập, project hoặc interview assignment, kết quả cuối cùng không phải là tất cả. Điều quan trọng hơn là thought process (quá trình tư duy) và lý do đằng sau việc lựa chọn giải pháp đó. 

Diễn giả cũng nhấn mạnh tầm quan trọng của việc biết đặt câu hỏi “why”, dám chất vấn lại cách hiểu của chính mình, đồng thời biết chấp nhận sai lầm để học hỏi từ đó. Khi bắt đầu bước vào môi trường làm việc thực tế, những yếu tố như integrity (sự chính trực), tư duy dài hạn, khả năng xây dựng network, kỹ năng teamwork và khả năng đặt câu hỏi chính là những chìa khóa quan trọng nhất.

#### Chị Thảo - Phương pháp BMAD: Đưa toàn bộ đội ngũ Agile & AI vào IDE của bạn

Phần chia sẻ này xoay quanh phương pháp BMAD và cách đưa quy trình phát triển phần mềm có hệ thống vào việc lập trình với AI. Việc "Vibe coding" (code tùy hứng với AI) có thể giúp tạo ra sản phẩm rất nhanh, nhưng nếu bỏ qua vòng đời phát triển phần mềm (software development lifecycle), chúng sẽ dễ tạo ra những đoạn code rác, khó maintain và khó kiểm soát.

Nếu chúng ta liên tục prompt AI mà thiếu đi tài liệu (documentation), thiếu kiến trúc (architecture) và không có kế hoạch rõ ràng, AI sẽ dần mất đi ngữ cảnh (context) và sinh ra code chắp vá. Để giải quyết vấn đề này, phương pháp BMAD giúp tổ chức quy trình một cách cấu trúc hơn.

Quy trình bao gồm các bước: brainstorming, tạo PRD (Product Requirements Document), tạo architecture document, phân chia epic/story, sau đó mới dùng đến các development agent và QA/review agent. Việc chia nhỏ task và sử dụng agent theo từng vai trò cụ thể giúp AI hiểu rõ context hơn, từ đó giảm thiểu tình trạng hallucination và giúp con người giữ được quyền kiểm soát tốt hơn đối với toàn bộ quá trình phát triển phần mềm. Bài học thực tế lớn nhất là: khi sử dụng AI để lập trình, tài liệu và quy trình rõ ràng cũng quan trọng không kém gì việc viết prompt.

### Những Gì Học Được

* **Thói quen học tập bền vững**: Hiểu được cơ chế hoạt động của não bộ để xây dựng môi trường và thói quen học tập hiệu quả, duy trì động lực lâu dài thay vì chỉ cố gắng nhất thời.
* **Prompt engineering và cách dùng AI hiệu quả**: Nắm vững các thành phần của một prompt chuẩn, các best practices và kỹ thuật nâng cao để tối ưu hóa chất lượng đầu ra từ các mô hình ngôn ngữ.
* **Foundation, thought process và khả năng đặt câu hỏi**: Nhận thức được tầm quan trọng của kiến thức nền tảng, tư duy giải quyết vấn đề và thái độ học hỏi không ngừng trong thời đại AI.
* **BMAD Method và tư duy phát triển phần mềm có hệ thống**: Nắm bắt cách kết hợp AI vào vòng đời phát triển phần mềm một cách có cấu trúc, sử dụng tài liệu và chia nhỏ tác vụ để kiểm soát chất lượng code.
* **Giá trị của cộng đồng FCAJ**: Nhận thấy lợi ích to lớn từ việc chia sẻ kiến thức, kinh nghiệm thực tế và mở rộng network với các chuyên gia trong ngành.

### Ứng Dụng Vào Quá Trình Internship

* Áp dụng thói quen học đều đặn để hoàn thành Worklog, cập nhật Event Participated và hoàn thiện các phần còn lại của báo cáo thực tập.
* Sử dụng các kỹ thuật prompt engineering rõ ràng, cấu trúc hơn khi học các dịch vụ AWS, viết tài liệu kỹ thuật, phân tích lỗi (debug) hoặc tìm hiểu một dịch vụ mới.
* Khi thực hiện các project hoặc workshop, không chỉ quan tâm đến kết quả cuối cùng mà còn phải rèn luyện thought process: hiểu rõ lý do lựa chọn kiến trúc, dịch vụ AWS tương ứng và workflow.
* Áp dụng phương pháp chia nhỏ task, chuẩn bị tài liệu kỹ càng và kiểm soát ngữ cảnh (context) khi dùng AI để hỗ trợ phát triển phần mềm.

### Trải nghiệm trong event

Đây là một trong những event đầu tiên tôi tham gia trong quá trình thực tập AWS FCAJ Cloud Intern. Sự kiện đã mang đến một trải nghiệm toàn diện khi kết hợp khéo léo giữa các yếu tố kỹ thuật và mindset: từ phương pháp học tập, cách ứng dụng AI, kiến thức cloud, prompt engineering, cho đến định hướng tư duy nghề nghiệp (career mindset) và phương pháp phát triển phần mềm.

Tôi đặc biệt ấn tượng với phần chia sẻ về Prompt Engineering và dự án thực tế Proptimizer, vì nội dung này liên quan trực tiếp và giải quyết đúng những khó khăn tôi đang gặp phải khi sử dụng AI trong học tập cũng như xử lý công việc kỹ thuật hàng ngày.

Thông qua buổi chia sẻ, tôi nhận ra rằng việc học cloud hay bất kỳ công nghệ nào không chỉ đơn thuần là việc học cách sử dụng công cụ. Để đi đường dài, tôi cần xây dựng một kiến thức nền tảng (foundation) thật vững chắc, rèn luyện tư duy đặt câu hỏi, phát triển khả năng học tập bền vững và áp dụng một phương pháp làm việc có hệ thống, kỷ luật.

### Một số hình ảnh khi tham gia sự kiện

![Mr. Long Huynh sharing session](/images/4-EventParticipated/4.1-Event1/long-session.png)
*Anh Long chia sẻ về thói quen học tập và động lực.*

![Mr. Thinh Nguyen sharing session](/images/4-EventParticipated/4.1-Event1/thinh-session.png)
*Anh Thịnh trình bày về Automated Prompt Engineering và dự án Proptimizer.*

![Mr. Khang Nguyen sharing session](/images/4-EventParticipated/4.1-Event1/khang-session.png)
*Anh Khang chia sẻ về kỹ năng và mindset cho fresher trong kỷ nguyên AI.*

![Ms. Thao sharing session](/images/4-EventParticipated/4.1-Event1/thao-session.png)
*Chị Thảo giới thiệu phương pháp BMAD trong phát triển phần mềm với AI.*

![Group photo at FCAJ Community Day](/images/4-EventParticipated/4.1-Event1/group-photo.png)
*Ảnh chụp tập thể tại sự kiện FCAJ Community Day.*