---
title: "Tuần 2 - Nền tảng & Phân đoạn Mạng VPC"
date: 2026-04-27
weight: 2
chapter: false
draft: false
pre: " <b> 1.2. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Dịch vụ trọng tâm | Amazon Virtual Private Cloud (VPC) |
| Mục tiêu chính | Xây dựng và xác thực nền tảng mạng AWS được phân đoạn |
| Region chính | `ap-southeast-1` - Asia Pacific (Singapore) |
| VPC CIDR | `10.10.0.0/16` |
| Public subnets | 2 |
| Private subnets | 2 |
| Availability Zones (AZ) | 2 |
| Truy cập Internet | Internet Gateway dành cho định tuyến public subnet |
| Triển khai Compute | Chuyển sang Tuần 3 |
| Công cụ vận hành | Chuyển sang Tuần 4 |
| Trạng thái trong tuần | Đã hoàn tất nền tảng VPC; một ảnh chụp màn hình liên kết route-table sẽ được hoàn thiện sau |

## Mục tiêu

Mục tiêu của Tuần 2 là thiết kế và xác thực một nền tảng mạng Amazon VPC cho dự án giám sát bảo mật FCAJ.

Bài lab tập trung vào việc quy hoạch dải mạng CIDR, phân đoạn subnet public/private, định tuyến qua Internet Gateway, hoạt động của route-table, và thiết lập đường ranh giới bằng Security Group. Các thành phần như EC2 (Compute), xác thực NAT Gateway, và các công cụ vận hành được chủ ý lùi lại vào các tuần sau để bài viết này có thể tập trung hoàn toàn vào kiến trúc mạng.

## Bối cảnh Dự án

Dự án Hybrid IDS/AI SOC cần các vùng mạng tách biệt dành cho các điểm truy cập công khai (public entry points), các dịch vụ backend, các trình xử lý AI (AI processing workers) và cơ sở dữ liệu (database) trong tương lai.

Bài lab VPC của Tuần 2 đã thiết lập một mô hình phân đoạn mạng để sau này có thể đặt các thành phần giao tiếp ra bên ngoài (public-facing) vào các public subnet, đồng thời giữ các tác vụ backend, AI và database an toàn bên trong các private subnet. Đây là một bài thực hành mạng trên AWS, không phải là đợt chuyển đổi (migration) cho dự án hoàn chỉnh trên production.

## Nhật ký công việc hàng ngày

| Ngày | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Ngày 1 | 27/04/2026 | Rà soát yêu cầu VPC và lên kế hoạch cho IPv4 CIDR | Dải địa chỉ VPC và các dải subnet đã được định nghĩa | Các dải CIDR cần không được trùng lặp và phải dễ nhận diện | Tạo VPC và các subnets |
| Ngày 2 | 29/04/2026 | Tạo VPC và 4 subnets trải rộng trên 2 Availability Zones | Bố cục mạng public/private đã được thiết lập | Tên của subnet không quyết định tính chất (public/private) của subnet đó | Cấu hình Internet Gateway và bảng định tuyến public (public route table) |
| Ngày 3 | 30/04/2026 | Đính kèm Internet Gateway và cấu hình định tuyến public | Đã xác thực việc định tuyến từ public subnet ra Internet Gateway | Chỉ đính kèm Internet Gateway là chưa đủ, cần phải cấu hình route-table | Rà soát lại việc liên kết route-table (route-table associations) |
| Ngày 4 | 02/05/2026 | Rà soát định tuyến private và ranh giới Security Group | Vùng mạng private đã được tách biệt khỏi định tuyến trực tiếp qua Internet Gateway | Security Groups kiểm soát quyền truy cập lưu lượng nhưng không làm cho một subnet trở thành public hay private | Tiếp tục với bài lab EC2 public/private ở Tuần 3 |

## Tổng quan Kiến trúc Mạng

VPC được thiết kế với một không gian địa chỉ private lớn và 4 dải subnet nhỏ hơn. Các public subnet được dành riêng cho các điểm kết nối đầu vào (như bastion host, load balancer, hoặc dashboard trong tương lai). Các private subnet được dành cho các tác vụ nội bộ như dịch vụ backend, AI workers, xử lý log, hoặc cơ sở dữ liệu.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e06-network-architecture.png" alt="Week 2 VPC network architecture">
  <figcaption>Hình 1 - Bài lab Tuần 2 phân tách các tuyến public khỏi vùng làm việc private bên trong một VPC duy nhất.</figcaption>
</figure>

| Ghi chú bằng chứng (Evidence note) | Chi tiết (Detail) |
| --- | --- |
| Cần kiểm tra gì (What to check) | VPC CIDR `10.10.0.0/16`, 2 public subnets, 2 private subnets, và ranh giới Internet Gateway |
| Chứng minh điều gì (What it proves) | Thiết kế của Tuần 2 tách biệt vùng mạng public với vùng private trước khi bắt đầu triển khai các máy chủ (compute) |
| Hạn chế (Limitation) | Đây chỉ là sơ đồ báo cáo; các ảnh chụp màn hình AWS Console bên dưới mới cung cấp bằng chứng triển khai thực tế |

## Thiết kế VPC và CIDR

VPC đã được tạo với dải CIDR:

```text
10.10.0.0/16
```

Dải này cung cấp một không gian địa chỉ đủ lớn cho các bài thực nghiệm về máy chủ, cơ sở dữ liệu và dịch vụ sau này, đồng thời giúp bài lab dễ dàng quản lý.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e01-vpc-cidr.png" alt="VPC CIDR configuration">
  <figcaption>Hình 2 - VPC được tạo với dải CIDR <code>10.10.0.0/16</code>; Account Owner ID của AWS đã được ẩn.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Trạng thái (State) VPC là `Available`, default VPC là `No`, và IPv4 CIDR là `10.10.0.0/16` |
| Chứng minh điều gì | Một VPC dành riêng đã được tạo cho mạng lab thay vì sử dụng VPC mặc định |
| Dữ liệu bị ẩn (Hidden data) | Chỉ có AWS account owner ID bị che; VPC ID và route table ID vẫn hiển thị rõ |

## Thiết kế Subnet Public và Private

Không gian địa chỉ VPC được chia thành 4 subnet `/24`:

| Subnet | Loại (Type) | CIDR | Mục đích |
| --- | --- | --- | --- |
| Public Subnet 1 | Public | `10.10.1.0/24` | Tài nguyên kết nối public trong tương lai |
| Public Subnet 2 | Public | `10.10.2.0/24` | Tài nguyên kết nối public ở một AZ khác |
| Private Subnet 1 | Private | `10.10.3.0/24` | Các tải công việc backend, AI, hoặc database |
| Private Subnet 2 | Private | `10.10.4.0/24` | Các tải công việc backend, AI, hoặc database ở một AZ khác |

Mỗi subnet được chụp màn hình riêng lẻ để người xem có thể đọc được tên subnet, subnet ID, trạng thái, VPC, dải CIDR và Availability Zone mà không bị mất chi tiết.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02a-public-subnet-1.png" alt="Public Subnet 1">
  <figcaption>Hình 3 - Public Subnet 1 được tạo làm subnet kết nối public đầu tiên với dải CIDR <code>10.10.1.0/24</code>.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Name là `Public Subnet 1`, state là `Available`, VPC được gán, CIDR `10.10.1.0/24` và Availability Zone |
| Chứng minh điều gì | Dải subnet public đầu tiên đã được tạo bên trong VPC Tuần 2 |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02b-public-subnet-2.png" alt="Public Subnet 2">
  <figcaption>Hình 4 - Public Subnet 2 được tạo làm subnet kết nối public thứ hai với dải CIDR <code>10.10.2.0/24</code>.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Name là `Public Subnet 2`, state là `Available`, VPC được gán, CIDR `10.10.2.0/24` và Availability Zone |
| Chứng minh điều gì | Subnet public thứ hai đã được chuẩn bị để hỗ trợ mô hình mạng đa vùng (multi-AZ) |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02c-private-subnet-1.png" alt="Private Subnet 1">
  <figcaption>Hình 5 - Private Subnet 1 được tạo cho các dịch vụ nội bộ với dải CIDR <code>10.10.3.0/24</code>.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Name là `Private Subnet 1`, state là `Available`, VPC được gán, CIDR `10.10.3.0/24` và Availability Zone |
| Chứng minh điều gì | Dải private subnet đầu tiên được dự phòng cho các workload như backend, AI hoặc cơ sở dữ liệu sau này |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e02d-private-subnet-2.png" alt="Private Subnet 2">
  <figcaption>Hình 6 - Private Subnet 2 được tạo cho các dịch vụ nội bộ với dải CIDR <code>10.10.4.0/24</code>.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Name là `Private Subnet 2`, state là `Available`, VPC được gán, CIDR `10.10.4.0/24` và Availability Zone |
| Chứng minh điều gì | Subnet private thứ hai đã được chuẩn bị để hỗ trợ xử lý dịch vụ nội bộ tại một Availability Zone khác |

Một subnet được phân loại là public khi bảng định tuyến (route table) liên kết với nó chứa một tuyến (route) đi đến Internet Gateway. Để một EC2 instance giao tiếp với internet thông qua IPv4, bản thân instance đó cũng cần một địa chỉ IPv4 public hoặc Elastic IP cùng với các quyền truy cập hợp lệ (traffic permissions).

## Bảng Định tuyến và Internet Gateway (Route Tables & IGW)

Một Internet Gateway (IGW) đã được tạo, đính kèm vào VPC và được dùng bởi public route table.

| Bảng định tuyến (Route table) | Đích đến (Destination) | Mục tiêu (Target) | Ý nghĩa |
| --- | --- | --- | --- |
| Public route table | `10.10.0.0/16` | `local` | Giao tiếp nội bộ trong VPC |
| Public route table | `0.0.0.0/0` | Internet Gateway | Tuyến ra Internet cho giao thức IPv4 của các public subnet |
| Private route table | `10.10.0.0/16` | `local` | Giao tiếp nội bộ trong VPC |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e03-igw-attached.png" alt="Internet Gateway attached to VPC">
  <figcaption>Hình 7 - Internet Gateway đã được đính kèm vào VPC Tuần 2; Account Owner ID của AWS đã được ẩn.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Trạng thái IGW là `Attached` và VPC ID trỏ đúng về VPC của Tuần 2 |
| Chứng minh điều gì | VPC hiện có một Internet Gateway để sẵn sàng phục vụ cho các public route-table |
| Dữ liệu bị ẩn | Chỉ có AWS account owner ID bị che; IGW ID và VPC ID vẫn hiển thị rõ |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e04-public-route-table.png" alt="Public route table">
  <figcaption>Hình 8 - Bảng định tuyến public chứa tuyến định tuyến nội bộ VPC và một tuyến mặc định (default route) trỏ về Internet Gateway.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Cả hai route: `10.10.0.0/16 -> local` và `0.0.0.0/0 -> Internet Gateway` đều ở trạng thái active (hoạt động) |
| Chứng minh điều gì | Các public subnet sẽ có đường kết nối IPv4 ra internet khi được liên kết (associate) với route table này |
| Hạn chế | Không có ảnh chụp màn hình cụ thể cho phần liên kết (route-table association), do đó tính liên kết này được ghi nhận từ lab notes |

Việc cấu hình để các private subnet có thể truy cập outbound ra internet thông qua NAT Gateway không nằm trong mục tiêu của Tuần 2. Nội dung này sẽ được thực hiện và ghi nhận ở Tuần 3, cùng với các bài kiểm thử máy chủ EC2 public/private.

## Phân đoạn bằng Security Group

Các Security Group (SG) đã được cấu hình đóng vai trò kiểm soát lưu lượng ở mức tài nguyên (resource-level) cho bài lab VPC. SG không quyết định xem một subnet là public hay private; bảng định tuyến quyết định đường đi (network path), trong khi Security Group quyết định liệu dòng traffic đó có được phép đi qua tài nguyên đính kèm hay không.

Các bộ quy tắc trong giai đoạn thực hành (lab-stage) bao gồm cả những quyền truy cập mở rộng nhằm mục đích học tập. Đây không phải là tư thế bảo mật (posture) trên môi trường production thực tế. Một phiên bản production cần giới hạn quyền truy cập quản trị đối với các Source IP tĩnh (known source IPs), sử dụng mô hình qua bastion host, hoặc các mô hình truy cập private được quản lý chặt chẽ.

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e05a-public-sg.png" alt="Public subnet Security Group">
  <figcaption>Hình 9 - Security Group của public subnet được tạo với mục đích thực hành kết nối SSH và ICMP; Account Owner ID của AWS đã được ẩn.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Tên Security Group, VPC ID, luật Inbound SSH, luật Inbound ICMP, và các dải Source IPs được mở rộng cho bài lab |
| Chứng minh điều gì | Các tài nguyên thực hành (lab) mang tính chất public-facing được kiểm soát qua một SG chuyên dụng thay vì chỉ qua tên gọi của subnet |
| Cảnh báo quan trọng | Các Source `0.0.0.0/0` chỉ là luật truy cập cho bài lab, không phải là cấu hình bảo mật chuẩn cho production |

<figure class="worklog-evidence">
  <img src="/images/worklog/week2/w2-e05b-private-sg.png" alt="Private subnet Security Group">
  <figcaption>Hình 10 - Các luật cho Security Group của private subnet đã được thiết lập để quản lý lưu lượng truy cập nội bộ có kiểm soát.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Source cho SSH đến từ public Security Group, Source cho ICMP đến từ dải `10.10.0.0/16`, và luật HTTPS cho bài lab |
| Chứng minh điều gì | Luồng truy cập của các dịch vụ bên trong private subnet đã được thiết kế riêng biệt so với public subnet |
| Cảnh báo quan trọng | Rule HTTPS (cho phép broad source) được giữ lại nhằm phục vụ khám phá lab và sẽ bị siết chặt lại khi triển khai production |

## Xác thực và Bằng chứng (Validation and Selected Evidence)

| ID | Bằng chứng | Kết quả |
| --- | --- | --- |
| W2-E01 | VPC và CIDR `10.10.0.0/16` | Đã xác minh |
| W2-E02 | Public Subnet 1, CIDR `10.10.1.0/24` | Đã xác minh |
| W2-E03 | Public Subnet 2, CIDR `10.10.2.0/24` | Đã xác minh |
| W2-E04 | Private Subnet 1, CIDR `10.10.3.0/24` | Đã xác minh |
| W2-E05 | Private Subnet 2, CIDR `10.10.4.0/24` | Đã xác minh |
| W2-E06 | Đính kèm Internet Gateway vào VPC | Đã xác minh |
| W2-E07 | Route public `0.0.0.0/0 -> IGW` | Đã xác minh |
| W2-E08 | Đường cơ sở Security Group public và private | Đã xác minh |
| W2-E09 | Việc liên kết Subnet với Route-table (Route-table associations) | Đã xem xét từ ghi chú bài lab; chưa chụp lại thành screenshot riêng biệt |

Các ghi chú lab thô và ảnh chụp màn hình toàn bộ giao diện không được nhúng trực tiếp trên trang Hugo. Thay vào đó, báo cáo này dùng các hình ảnh có chọn lọc được lưu trong đường dẫn `/images/worklog/week2/`.

## Vấn đề và Khắc phục (Issues and Troubleshooting)

| Vấn đề / Quyết định | Giải pháp |
| --- | --- |
| Tên gọi của subnet rất dễ gây hiểu nhầm | Đã ghi chú lại rằng việc liên kết với bảng định tuyến (route-table association) mới là thứ quyết định một mạng là public hay private |
| Đính kèm Internet Gateway là chưa đủ để tạo được định tuyến public (public routing) | Đã thêm và xác thực route mặc định trỏ về Internet Gateway |
| Route Tables và Security Groups có vẻ bị chồng chéo về khái niệm | Đã phân tách vai trò của chúng: Route table chọn đường đi, Security Groups cho phép/từ chối luồng dữ liệu |
| Hình ảnh bằng chứng cho bài lab tổng hợp đôi khi bao gồm cả EC2, NAT, và tool vận hành | Chỉ giới hạn báo cáo Tuần 2 ở việc thiết lập VPC foundation và dời các phần kia sang Tuần 3, Tuần 4 |
| Vài ảnh chụp màn hình có dính thông tin định danh tài khoản AWS | Đã che ID chủ tài khoản (Account owner IDs) đồng thời giữ nguyên IDs của tài nguyên (resource IDs) nhằm làm bằng chứng xác thực |

## Bài học rút ra (Lessons Learned)

Việc đặt tên cho subnet (như thêm chữ "public" hay "private") là không đủ. Tính chất public hay private thực sự phụ thuộc vào các bảng định tuyến (route tables), liên kết subnet (subnet associations), việc cấp phát địa chỉ IP public, và quyền truy cập dữ liệu mạng (traffic permissions).

Một bài học quan trọng khác là kiến trúc VPC phải được xem xét dưới góc nhìn tổng thể của một hệ thống. Hoạch định mạng CIDR, sơ đồ subnet, cấu hình kết nối Internet Gateway, các bảng định tuyến, và Security Groups - tất cả phải vận hành trơn tru cùng nhau. Nếu chỉ cần thiết lập sai một trong các yếu tố trên, một tài nguyên cũng có thể mất kết nối hoặc vô tình bị lộ ra ngoài internet.

## Kết quả đạt được trong tuần

Vào cuối Tuần 2, dự án đã có một nền tảng mạng AWS VPC được xác thực thành công, với một dải mạng CIDR quy chuẩn, 4 subnets, hệ thống định tuyến public qua Internet Gateway, phân đoạn được subnet private, cùng với đường cơ sở về Security Group.

Mô hình mạng này đã sẵn sàng cho Tuần 3 - nơi sẽ tập trung triển khai và tài liệu hóa máy chủ EC2 public/private, kiểu truy cập qua bastion host, cơ chế NAT Gateway, và tính kết nối internet từ các subnet private.

## Kế hoạch tuần tiếp theo

Tuần 3 sẽ tập trung vào việc triển khai các máy chủ (compute) lên hệ thống VPC vừa tạo ở Tuần 2. Trọng tâm sẽ là:

- Các phiên bản EC2 Public
- Các phiên bản EC2 Private
- Hình thức truy cập SSH thông qua bastion
- Thiết lập NAT Gateway và Elastic IP
- Đánh giá quyền truy cập internet (outbound) từ private subnet
- Cân nhắc chi phí khi dùng NAT và việc dọn dẹp tài nguyên thừa
