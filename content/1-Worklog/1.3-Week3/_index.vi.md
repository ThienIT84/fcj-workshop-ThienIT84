---
title: "Tuần 3 - EC2, Truy cập Bastion & NAT Private Egress"
date: 2026-05-25
weight: 3
chapter: false
draft: false
pre: " <b> 1.3. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Dịch vụ trọng tâm | Amazon EC2 |
| Dịch vụ hỗ trợ | Key Pair, Security Group, Elastic IP, NAT Gateway, Route Table |
| Mục tiêu chính | Triển khai compute public/private và xác thực mô hình truy cập kiểu bastion |
| Region chính | `ap-southeast-1` - Asia Pacific (Singapore) |
| Nguồn VPC | Nền tảng VPC từ Tuần 2 |
| Mô hình workload public | Bastion / jump host |
| Mô hình workload private | Backend, AI worker, hoặc bộ xử lý log (log processor) |
| Công cụ vận hành | Chuyển sang Tuần 4 |
| Trạng thái trong tuần | Đã triển khai; đang chờ hoàn thiện nhật ký thời gian và rà soát bằng chứng thành công của NAT |

## Mục tiêu

Mục tiêu của Tuần 3 là triển khai các tài nguyên compute vào nền tảng mạng đã tạo ở Tuần 2, đồng thời xác thực mô hình truy cập cơ bản theo kiểu public/private.

Bài lab tập trung vào Amazon EC2, key pair, Security Group, truy cập SSH kiểu bastion, cấp phát Elastic IP, định tuyến qua NAT Gateway và thiết kế outbound cho private subnet. Các công cụ vận hành như Session Manager, EC2 Instance Connect Endpoint, VPC Flow Logs, Reachability Analyzer và CloudWatch được chủ ý lùi lại sang Tuần 4.

## Bối cảnh Dự án

Dự án Hybrid IDS/AI SOC không nên để các thành phần backend, xử lý AI hay log-processing tiếp xúc trực tiếp với internet. Mô hình an toàn hơn là đặt các tài nguyên kết nối công khai (public entry resources) vào public subnet và giữ các thành phần quan trọng trong private subnet.

Trong bài lab này, EC2 public đóng vai trò là một bastion hoặc điểm kết nối được kiểm soát. EC2 private đóng vai trò là một backend, AI worker, hoặc bộ xử lý log nội bộ trong tương lai. NAT Gateway được xem xét như một con đường outbound cho các tài nguyên private cần tải package hoặc gọi external service, mà không cần nhận lưu lượng internet từ bên ngoài đi vào.

## Nhật ký công việc hàng ngày

| Ngày | Ngày tháng | Thời lượng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- | --- |
| Ngày 1 | 25/05/2026 | Confirm before publishing | Rà soát VPC Tuần 2, các public subnet, private subnet và mô hình Security Group | Đã xác định kế hoạch đặt các EC2 instance | Các workload quan trọng nên được giữ trong private subnet | Khởi chạy EC2 public và private |
| Ngày 2 | 25/05/2026 | Confirm before publishing | Tạo EC2 public và EC2 private với key pair và Security Group | Mô hình compute public/private đã được chuẩn bị | EC2 public có thể hoạt động như điểm kết nối bastion | Kiểm tra việc truy cập vào private instance |
| Ngày 3 | 25/05/2026 | Confirm before publishing | Kết nối từ EC2 public sang EC2 private thông qua địa chỉ IP nội bộ (private IP) | Đã xác thực được mô hình truy cập SSH kiểu jump-host | EC2 private không cần địa chỉ IPv4 public | Cấu hình đường outbound cho private subnet |
| Ngày 4 | 25/05/2026 | Confirm before publishing | Cấp phát Elastic IP, tạo tài nguyên NAT Gateway và rà soát bảng định tuyến private | Đường định tuyến từ private subnet đến NAT Gateway đã được cấu hình | NAT Gateway gây phát sinh chi phí và cần được xóa sau khi hoàn thành bài lab | Xem xét kết quả outbound từ private subnet |
| Ngày 5 | 25/05/2026 | Confirm before publishing | Rà soát kết quả kiểm thử outbound từ private subnet và ghi chép hành vi ping chưa được giải quyết | Ghi nhận ghi chú xử lý sự cố (troubleshooting) mà không phóng đại kết quả | `ping amazon.com` trả về mất gói (packet loss) trong bằng chứng hiện có | Chụp lại bài kiểm thử outbound thành công nếu lặp lại sau này |

## Thiết kế Compute EC2 Public và Private

Thiết kế compute của Tuần 3 dùng hai vai trò EC2:

| Thành phần | Vị trí | Mục đích |
| --- | --- | --- |
| EC2 Public | Public subnet | Điểm kết nối SSH và jump host kiểu bastion |
| EC2 Private | Private subnet | Mô hình workload nội bộ: backend, AI worker, hoặc log processor |

Instance public cần một địa chỉ IPv4 public, định tuyến public subnet, một Security Group cho phép kết nối SSH được phê duyệt, và một key pair hợp lệ. Instance private không nên được truy cập trực tiếp từ internet; việc truy cập phải đi qua EC2 public hoặc một phương thức truy cập private được quản lý.

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e01-ec2-public-private.png" alt="Thông tin EC2 public instance">
  <figcaption>Hình 1 - Bằng chứng phía EC2 public cho thấy một instance đang chạy với thuộc tính public IPv4 và private IPv4.</figcaption>
</figure>

| Ghi chú bằng chứng (Evidence note) | Chi tiết (Detail) |
| --- | --- |
| Cần kiểm tra gì (What to check) | Trạng thái instance, địa chỉ public IPv4, địa chỉ private IPv4 và public DNS |
| Chứng minh điều gì (What it proves) | Phía EC2 public của mô hình compute đã sẵn sàng để kiểm thử truy cập |
| Hạn chế (Limitation) | Ảnh này tập trung vào các thuộc tính truy cập của EC2 public; vị trí của EC2 private được xác thực thông qua bằng chứng SSH qua bastion ở phần bên dưới |

## Luồng truy cập qua Bastion (Bastion Access Flow)

Private instance được truy cập từ EC2 public thông qua địa chỉ private IP của instance đó. Mô hình này xác thực luồng truy cập kiểu jump-host:

```text
Laptop
  -> EC2 Public / Bastion
  -> EC2 Private thông qua private IP
```

Mô hình này phù hợp hơn so với việc để một private workload tiếp xúc trực tiếp với internet, vì nó thu hẹp đường vào quản trị (administrative entry path).

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e02-bastion-private-ssh.png" alt="Kết nối SSH từ Bastion vào EC2 private">
  <figcaption>Hình 2 - SSH từ EC2 public (bastion) vào EC2 private thành công thông qua private IP.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Lệnh SSH từ `ip-10-10-1-182` đến `10.10.4.240`, màn hình đăng nhập Amazon Linux, lệnh `whoami` và hostname của private instance |
| Chứng minh điều gì | EC2 public có thể tiếp cận và quản trị EC2 private thông qua mạng private nội bộ của VPC |
| Ghi chú quan trọng (Important note) | Kết quả `ping amazon.com` xuất hiện ở cuối terminal trong cùng ảnh chụp được coi là bằng chứng xử lý sự cố (troubleshooting evidence), không phải bằng chứng NAT thành công |

## Thiết kế NAT Gateway và Private Egress

Các private instance thường cần quyền truy cập internet chiều ra (outbound) để tải bản cập nhật gói phần mềm hoặc gọi các API bên ngoài. Chúng không nên nhận trực tiếp lưu lượng internet từ bên ngoài vào. NAT Gateway hỗ trợ mô hình này bằng cách cho phép lưu lượng outbound từ private subnet đi qua public subnet và Internet Gateway.

Bài lab đã chuẩn bị đường truyền này sử dụng:

| Thành phần | Vai trò |
| --- | --- |
| Elastic IP | Địa chỉ IPv4 public được dùng bởi NAT Gateway |
| NAT Gateway | Đường internet chiều ra (outbound) cho các private subnet |
| Private route table | Định tuyến lưu lượng `0.0.0.0/0` đến NAT Gateway |

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e03-elastic-ip-for-nat.png" alt="Elastic IP được cấp phát cho NAT Gateway">
  <figcaption>Hình 3 - Một Elastic IP đã được cấp phát dùng cho NAT Gateway phục vụ đường private egress.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Thông báo cấp phát Elastic IP thành công, tên EIP, địa chỉ IPv4 đã cấp phát và allocation ID |
| Chứng minh điều gì | Một tài nguyên IPv4 public tĩnh đã được chuẩn bị cho NAT Gateway |
| Ghi chú chi phí (Cost note) | Tài nguyên Elastic IP và NAT Gateway có thể phát sinh chi phí nếu để chạy sau khi hoàn thành bài lab |

<figure class="worklog-evidence">
  <img src="/images/worklog/week3/w3-e04-private-route-to-nat.png" alt="Bảng định tuyến private trỏ đến NAT Gateway">
  <figcaption>Hình 4 - Bảng định tuyến private chuyển hướng lưu lượng outbound mặc định đến NAT Gateway; Account Owner ID của AWS đã được ẩn.</figcaption>
</figure>

| Ghi chú bằng chứng | Chi tiết |
| --- | --- |
| Cần kiểm tra gì | Bảng định tuyến private, liên kết subnet, route `10.10.0.0/16 -> local`, và route `0.0.0.0/0 -> NAT Gateway` |
| Chứng minh điều gì | Định tuyến outbound của private subnet đã được cấu hình qua NAT thay vì đi thẳng ra Internet Gateway |
| Dữ liệu bị ẩn | Chỉ có AWS account owner ID bị che; các ID của route table, subnet, VPC và NAT target vẫn hiển thị rõ |

## Các quyết định về Bảo mật (Security Decisions)

| Quyết định | Lý do |
| --- | --- |
| Giữ các workload nội bộ trong private subnet | Backend, AI worker và log processor không nên tiếp xúc trực tiếp với internet |
| Dùng EC2 public như một điểm kết nối kiểu bastion | Tập trung đường vào quản trị thay vì mở truy cập trực tiếp vào các tài nguyên private |
| Dùng Security Group như kiểm soát cấp tài nguyên | Route table xác định đường truyền mạng; Security Group xác định lưu lượng nào được phép |
| Dùng NAT Gateway cho thiết kế outbound từ private subnet | Tài nguyên private có thể khởi tạo kết nối internet chiều ra mà không cần nhận lưu lượng inbound từ internet |
| Lùi các công cụ vận hành được quản lý sang Tuần 4 | SSM, EIC Endpoint, Flow Logs, Reachability Analyzer và CloudWatch xứng đáng có một bài worklog riêng tập trung vào vận hành |

## Xác thực và Bằng chứng (Validation and Selected Evidence)

| ID | Bằng chứng | Kết quả |
| --- | --- | --- |
| W3-E01 | Các thuộc tính truy cập của EC2 public instance | Đã xác minh |
| W3-E02 | SSH từ EC2 public vào EC2 private qua bastion | Đã xác minh |
| W3-E03 | Elastic IP được cấp phát cho NAT Gateway | Đã xác minh |
| W3-E04 | Route mặc định của private route table trỏ đến NAT Gateway | Đã xác minh |
| W3-E05 | Thành công kết nối internet outbound từ private subnet | Chưa xác nhận được từ bằng chứng hiện tại |

Các ghi chú lab thô và ảnh chụp màn hình toàn bộ giao diện không được nhúng trực tiếp trên trang Hugo. Báo cáo này dùng các hình ảnh có chọn lọc được lưu trong đường dẫn `/images/worklog/week3/`.

## Vấn đề và Khắc phục (Issues and Troubleshooting)

| Vấn đề / Quyết định | Giải pháp |
| --- | --- |
| EC2 private không nên tiếp xúc trực tiếp với internet | Dùng EC2 public như đường truy cập kiểu bastion |
| Quyền truy cập file private key rất nghiêm ngặt đối với SSH | Dùng SSH xác thực bằng key và tuân theo mô hình quyền truy cập private key đúng chuẩn |
| `ping amazon.com` cho kết quả `100% packet loss` trong bằng chứng terminal hiện có | Ghi chép như xử lý sự cố (troubleshooting) thay vì khẳng định NAT thành công; một bài kiểm thử trong tương lai nên dùng `curl`, `yum` hoặc một bài kiểm thử outbound HTTP/DNS khác |
| NAT Gateway và Elastic IP có thể phát sinh chi phí | Xem tài nguyên NAT như tài nguyên bài lab và dọn dẹp sau khi hoàn thành xác thực |
| Bằng chứng EC2 bao gồm thông tin vận hành nhạy cảm | Ảnh công khai vẫn giữ các technical ID hiển thị nhưng tránh để lộ AWS account owner ID |

## Bài học rút ra (Lessons Learned)

Public subnet không chỉ là nơi chạy workload. Nó còn có thể là một vùng kết nối đầu vào được kiểm soát (controlled entry zone) phục vụ mục đích quản trị, truy cập bastion, hoặc làm nơi đặt load balancer trong tương lai.

Compute trong private subnet là mặc định tốt hơn cho các thành phần backend và AI, vì instance đó không cần địa chỉ IPv4 public. Việc truy cập phải được thực hiện thông qua các đường có kiểm soát như bastion hoặc các công cụ vận hành AWS được quản lý.

NAT Gateway giải quyết một vấn đề khác so với bastion. Bastion giúp người quản trị tiếp cận các private instance; NAT giúp các private instance khởi tạo kết nối internet chiều ra (outbound). Hai tính năng này không nên bị nhầm lẫn với nhau.

## Kết quả đạt được trong tuần

Vào cuối Tuần 3, dự án đã có mô hình compute EC2 public/private được tài liệu hóa, kết nối với nền tảng VPC của Tuần 2. Truy cập SSH kiểu bastion vào private instance đã được xác thực, và định tuyến qua NAT Gateway đã được chuẩn bị để phục vụ kết nối internet outbound cho private subnet.

Bằng chứng hiện có chưa chứng minh được sự thành công của kết nối internet outbound từ private subnet, vì vậy bài viết này ghi nhận trung thực cấu hình route NAT và hành vi ping chưa được giải quyết.

## Kế hoạch tuần tiếp theo

Tuần 4 sẽ tập trung vào bảo mật vận hành và khả năng quan sát (observability) cho hạ tầng:

- AWS Systems Manager Session Manager
- EC2 Instance Connect Endpoint
- VPC Flow Logs
- Reachability Analyzer
- CloudWatch monitoring và alerting
