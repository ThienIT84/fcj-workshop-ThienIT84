---
title: "Tuần 1 - Bảo mật Tài khoản AWS & Đường cơ sở Truy cập IAM"
date: 2026-04-20
weight: 1
chapter: false
draft: false
pre: " <b> 1.1. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Dịch vụ trọng tâm | AWS Identity and Access Management (IAM) |
| Mục tiêu chính | Bảo mật tài khoản AWS và tạo quyền truy cập cho từng thành viên trong nhóm |
| Đầu ra chính | Bảo vệ Root, người dùng IAM quản trị (admin IAM user), các nhóm IAM, và người dùng cho các thành viên |
| Region chính | `ap-southeast-1` - Asia Pacific (Singapore) |
| Mô hình tài khoản | Tài khoản AWS độc lập (Standalone AWS account) |
| Ngày bắt đầu | 20/04/2026 |
| Ngày hoàn thành | 26/04/2026 |
| Trạng thái Migration | Chưa bắt đầu |
| Trạng thái trong tuần | Đã hoàn thành triển khai; đã thêm các bằng chứng công khai được chọn lọc |

## Mục tiêu

Mục tiêu của Tuần 1 là thiết lập một tài khoản AWS an toàn và đường cơ sở truy cập IAM (IAM access baseline) trước khi tạo hạ tầng đám mây cho dự án FCAJ.

Tuần này tập trung vào việc bảo vệ người dùng root, nhận thức về chi phí (billing), lựa chọn Region, tạo người dùng IAM cá nhân, phân quyền dựa trên nhóm (group-based permissions), và mô hình truy cập thực tế cho một nhóm thực tập sinh nhỏ. Tài khoản hiện tại vẫn là tài khoản độc lập; AWS Organizations và AWS Control Tower chưa được kích hoạt trong giai đoạn này.

Trong worklog này, **lightweight landing zone** (landing zone gọn nhẹ) có nghĩa là một đường cơ sở về bảo mật, định danh, Region và chi phí cho một tài khoản duy nhất. Nó không mang ý nghĩa là một landing zone đa tài khoản bằng AWS Control Tower.

![Root security baseline](/images/worklog/week1/w1-e01-root-security.png)

*Hình 1 - Trạng thái bảo vệ bằng MFA cho tài khoản root và các access-key của root đã được kiểm tra trước khi tạo tài nguyên dự án.*

## Nhật ký công việc hàng ngày

| Ngày | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Ngày 1 | 20/04/2026 | Kiểm tra bảo mật tài khoản root, trạng thái MFA của root, trạng thái access-key của root, credit tài khoản và chọn Region chính | Việc bảo vệ tài khoản root và các ràng buộc của tài khoản đã được nắm rõ trước khi tạo tài nguyên dự án | Root không nên được sử dụng cho công việc hàng ngày; `ap-southeast-1` được chọn làm Region chính | Tạo một IAM identity (định danh IAM) quản trị |
| Ngày 2 | 21/04/2026 | Cấu hình account alias/luồng đăng nhập IAM, tạo `thien-admin`, và chuẩn bị nhóm `FCAJ-Admins` | Công việc quản trị hàng ngày có thể được tách biệt khỏi người dùng root | Việc quản trị nên sử dụng một định danh IAM thay vì dùng chung thông tin đăng nhập root | Thiết kế các nhóm IAM dựa trên vai trò cho team |
| Ngày 3 | 23/04/2026 | Thiết kế các nhóm IAM, tạo các nhóm cho builder, và cấu hình chính sách mật khẩu tài khoản (password policy) | Một mô hình truy cập dựa trên nhóm đã được chuẩn bị cho các trách nhiệm liên quan đến cloud, backend, AI và frontend | Các chính sách bắt buộc dùng MFA (Forced-MFA) và tự phục vụ MFA (self-service MFA) được hoãn lại để giữ cho Tuần 1 mang tính thực tế và dễ dàng | Thêm các quyền truy cập chung dành cho giai đoạn học tập |
| Ngày 4 | 24/04/2026 | Tạo nhóm `FCAJ-Base-Users` và gán quyền `ViewOnlyAccess` được quản lý bởi AWS | Các thành viên có thể kiểm tra tài nguyên AWS trong giai đoạn học tập mà không có quyền triển khai (deployment permissions) | `ViewOnlyAccess` là quyền chỉ đọc (view-only) rộng rãi dành cho việc học tập, không phải là mô hình least-privilege (đặc quyền tối thiểu) cuối cùng | Tạo các người dùng IAM cho từng thành viên và gán vào các nhóm |
| Ngày 5 | 26/04/2026 | Tạo người dùng IAM cho thành viên, gán tư cách thành viên nhóm, và tránh tạo access keys cho những người dùng chỉ truy cập Console | Quyền truy cập của nhóm đã được chuẩn bị thông qua các định danh cá nhân thay vì chia sẻ chung thông tin đăng nhập | Truy cập Console là đủ cho giai đoạn học tập, vì vậy không có access keys dài hạn nào được tạo cho các người dùng thành viên | Chuyển sang nền tảng VPC trong Tuần 2 |

## Tóm tắt Triển khai Kỹ thuật

Quá trình triển khai trong Tuần 1 bắt đầu từ một vấn đề đơn giản: dự án có một tài khoản AWS và credit, nhưng nhóm không thể chia sẻ an toàn thông tin đăng nhập của người dùng root. Do đó, công việc chuyển từ việc bảo vệ tài khoản sang phân quyền truy cập cho từng cá nhân.

Đường cơ sở truy cập cuối cùng (access baseline) là:

- Người dùng root được bảo vệ và dành riêng cho các tác vụ cấp tài khoản (account-level), khôi phục và các trường hợp khẩn cấp.
- `thien-admin` được sử dụng cho các công việc quản trị tài khoản thường xuyên.
- Nhóm `FCAJ-Admins` nắm giữ quyền quản trị cho người được chỉ định.
- Nhóm `FCAJ-Base-Users` cung cấp quyền truy cập chỉ đọc rộng rãi cho giai đoạn học tập thông qua `ViewOnlyAccess` (được quản lý bởi AWS).
- Các nhóm builder (Role builder groups) mô tả các trách nhiệm liên quan đến cloud, backend, AI và frontend.
- Các nhóm builder không nhận được quyền triển khai (deployment permissions) trong Tuần 1.
- Các người dùng trong nhóm nhận được định danh IAM riêng biệt thay vì dùng chung thông tin đăng nhập.
- Người dùng truy cập Console không nhận được access keys trừ khi có một tác vụ sau này yêu cầu truy cập lập trình một cách rõ ràng.

![Administrative IAM identity](/images/worklog/week1/w1-e02-admin-identity.png)

*Hình 2 - Việc quản trị thường xuyên được thực hiện thông qua thien-admin thay vì người dùng root.*

Quyền triển khai được cố ý trì hoãn. Chúng chỉ nên được cấp cho một số thành viên nhất định trong giai đoạn nước rút di chuyển hệ thống lên AWS (AWS MVP migration sprint) vào Tuần 11 và sẽ được xem xét lại sau khi hoàn tất triển khai.

![View-only permission baseline](/images/worklog/week1/w1-e04-viewonly-baseline.png)

*Hình 3 - FCAJ-Base-Users nhận quyền truy cập chỉ đọc rộng rãi dành cho việc học thông qua ViewOnlyAccess.*

## Cấu trúc IAM

### Người dùng IAM (IAM Users)

| Người dùng IAM | Trách nhiệm |
| --- | --- |
| `thien-admin` | Quản trị tài khoản và trưởng nhóm (Team leader) |
| `fcaj-cloud` | Hạ tầng đám mây và mạng (Cloud infrastructure and networking) |
| `fcaj-backend` | Ứng dụng backend và triển khai (Backend application and deployment) |
| `fcaj-ai` | Tập dữ liệu (Dataset), các mô hình AI và trình phát hiện (detection worker) |
| `fcaj-frontend` | Triển khai frontend, giám sát và báo cáo (Frontend deployment, monitoring, and reporting) |

![Team IAM users](/images/worklog/week1/w1-e05-team-users.png)

*Hình 4 - Các thành viên dự án sử dụng định danh IAM riêng biệt thay vì chia sẻ thông tin đăng nhập.*

### Các nhóm IAM (IAM Groups)

| Nhóm IAM | Mục đích trong Tuần 1 |
| --- | --- |
| `FCAJ-Admins` | Các hoạt động quản trị tài khoản |
| `FCAJ-Base-Users` | Quyền truy cập chung cho giai đoạn học tập thông qua `ViewOnlyAccess` |
| `FCAJ-Cloud-Builders` | Nhóm vai trò về hạ tầng đám mây, chưa có quyền triển khai |
| `FCAJ-Backend-Builders` | Nhóm vai trò về Backend, chưa có quyền triển khai |
| `FCAJ-AI-Builders` | Nhóm vai trò về AI/Dữ liệu, chưa có quyền triển khai |
| `FCAJ-Frontend-Builders` | Nhóm vai trò về Frontend, chưa có quyền triển khai |

![IAM group structure](/images/worklog/week1/w1-e03-group-structure.png)

*Hình 5 - Các nhóm IAM giúp phân tách công việc quản trị, quyền truy cập học tập chung, và các trách nhiệm dự án.*

Các chính sách quyền (permissions) được đính kèm vào các nhóm thay vì đính kèm trực tiếp cho từng người dùng. Điều này giúp tài khoản dễ dàng được đánh giá lại và giữ cho mô hình quyền luôn linh hoạt khi dự án chuyển từ các bài thực hành sang công việc triển khai thực tế.

## Xác thực và Bằng chứng

Các ảnh chụp màn hình gốc và ghi chú triển khai được lưu giữ như bằng chứng nội bộ riêng tư. Báo cáo được công khai sẽ sử dụng các ảnh chụp màn hình được cắt và chọn lọc kỹ lưỡng, đảm bảo an toàn thông tin thay vì các hình ảnh chụp toàn bộ console gốc.

| ID | Bằng chứng | Kết quả |
| --- | --- | --- |
| W1-E01 | Đường cơ sở bảo mật của root (Root security baseline) | Đã xác minh |
| W1-E02 | Định danh IAM quản trị (Administrative IAM identity) | Đã xác minh |
| W1-E03 | Cấu trúc nhóm IAM | Đã xác minh |
| W1-E04 | Đường cơ sở quyền chỉ đọc (View-only permission baseline) | Đã xác minh |
| W1-E05 | Người dùng IAM của team | Đã xác minh |
| W1-E06 | Thành viên nhóm và việc phân bổ nhóm | Đã xác minh |

![Team group membership](/images/worklog/week1/w1-e06-membership.png)

*Hình 6 - Người dùng thành viên được cấp quyền thông qua các nhóm IAM thay vì gán quyền trực tiếp.*

## Vấn đề và Khắc phục (Issues and Troubleshooting)

| Vấn đề / Quyết định | Giải pháp |
| --- | --- |
| Thông tin đăng nhập root không nên được chia sẻ với các thành viên trong nhóm | Tạo `thien-admin` và các người dùng IAM thành viên để truy cập hàng ngày |
| Các credit của AWS Free Tier đã được kích hoạt | Giữ tài khoản ở dạng độc lập (standalone) và không kích hoạt AWS Organizations hay AWS Control Tower trong Tuần 1 |
| Chính sách bắt buộc cấu hình MFA làm tăng độ phức tạp trong lần giới thiệu đầu tiên | Trì hoãn chính sách JSON Forced-MFA và chính sách tự phục vụ MFA; người dùng đang hoạt động sẽ đăng ký MFA thủ công trong quá trình bắt đầu (onboarding) |
| `ViewOnlyAccess` rộng hơn so với mô hình least-privilege thực tế trên production | Chỉ sử dụng quyền này làm quyền truy cập đọc trong giai đoạn học tập và ghi chú rõ có thể thu hẹp sau này |
| Các nhóm builder có thể vô tình bao hàm các quyền triển khai (deployment access) | Đã tạo các nhóm này cho cấu trúc vai trò, nhưng không đính kèm bất kỳ quyền triển khai nào trong Tuần 1 |
| Bằng chứng có thể làm lộ thông tin nhạy cảm của AWS | Giữ bí mật các ảnh chụp màn hình gốc và chỉ xuất bản những bằng chứng đã được cắt gọt an toàn |

## Bài học rút ra (Lessons Learned)

Bảo mật AWS bắt đầu trước cả khi bất kỳ hạ tầng nào được tạo. Một VPC, EC2 instance, database hay storage bucket không nên được tạo ra cho đến khi tài khoản có một định danh rõ ràng và mô hình truy cập cụ thể.

Quản lý truy cập dựa trên nhóm sẽ dễ dàng được rà soát hơn so với việc gán trực tiếp chính sách (policies) cho từng người dùng. Nó cũng giúp cho việc cấp quyền triển khai tạm thời trong đợt chạy nước rút (sprint) chuyển đổi AWS MVP ở Tuần 11 trở nên dễ dàng hơn, và có thể thu hồi nhanh chóng sau khi hoàn tất.

Một bài học khác là ảnh chụp màn hình chỉ là bằng chứng, không phải bản thân worklog. Worklog nên giải thích những gì đã làm, những gì đã thay đổi, những quyết định nào đã được đưa ra, và những gì cần hoàn thiện trước khi xuất bản.

## Kết quả đạt được trong tuần

Vào cuối Tuần 1, dự án đã có một đường cơ sở truy cập bảo mật tài khoản AWS. Nhóm đã dành riêng tài khoản root cho các tác vụ cấp tài khoản, khôi phục và khẩn cấp; các thao tác quản trị thường xuyên được chuyển giao cho `thien-admin`; các định danh thành viên nhóm được phân tách dựa trên trách nhiệm công việc; và các quyền truy cập học tập chung được quản lý thống nhất thông qua các nhóm IAM.

Tài khoản hiện đã sẵn sàng cho các bài thực hành dịch vụ AWS tiếp theo, trong khi quyền triển khai vẫn được chủ ý trì hoãn cho đến sprint chuyển đổi AWS MVP.

## Kế hoạch tuần tiếp theo

Tuần 2 sẽ xây dựng nền tảng mạng AWS (AWS network foundation) cho dự án. Trọng tâm sẽ là:

- Thiết kế VPC
- Các subnet public và private
- Route tables
- Internet Gateway
- Security Groups
- Phân đoạn mạng dành cho backend, bộ máy AI (AI engine), và database workloads sau này

Điều này đưa dự án chuyển đổi từ bảo mật tài khoản sang thiết kế mạng trên đám mây.
