---
title: "Tuần 4 - Vận hành An toàn & Khả năng Quan sát"
date: 2026-05-11
weight: 4
chapter: false
draft: false
pre: " <b> 1.4. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Dịch vụ trọng tâm | AWS Systems Manager và Amazon CloudWatch |
| Dịch vụ hỗ trợ | EC2 Instance Connect Endpoint, VPC Flow Logs, IAM, CloudWatch Logs |
| Mục tiêu chính | Cải thiện quyền truy cập hạ tầng riêng (private), thu thập dữ liệu đo đạc (telemetry) và giám sát vận hành |
| Region chính | `ap-southeast-1` - Asia Pacific (Singapore) |
| Nguồn hạ tầng | VPC từ Tuần 2, mô hình EC2 từ Tuần 3, và môi trường SOC MVP migration đang hoạt động |
| Session Manager | Đã triển khai và xác minh |
| Truy cập EIC qua private-IP | Đã triển khai và xác minh |
| VPC Flow Logs | Đã chuyển các bản ghi đến CloudWatch Logs |
| Metric filter | Đã triển khai để theo dõi lưu lượng mạng bị từ chối (rejected) |
| CloudWatch alarm | Đã triển khai và kích hoạt; trạng thái cuối cùng ghi lại là `ALARM` |
| Reachability Analyzer | Hoãn lại |
| Trạng thái trong tuần | Triển khai hoàn thành với bằng chứng công khai được chọn lọc |

## Lưu ý Tái xác nhận Bằng chứng

Ghi chú gốc của lab vận hành Tuần 4 được tạo vào ngày `26/05/2026`.

Việc thu thập bằng chứng bổ sung và xác minh các cấu hình còn thiếu được hoàn thành vào ngày `30/06/2026` bằng cách sử dụng môi trường SOC MVP migration đang hoạt động. Điều này là cần thiết vì một số tài nguyên lab gốc đã bị xóa để kiểm soát chi phí, và một số ảnh chụp màn hình gốc không đủ mạnh để đưa vào worklog FCAJ cuối cùng.

Các ảnh chụp màn hình mới hơn nên được hiểu là bằng chứng tái xác nhận, không phải là khẳng định rằng mọi ảnh chụp màn hình Tuần 4 đều được chụp vào ngày lab gốc.

## Mục tiêu

Mục tiêu của Tuần 4 là cải thiện quyền truy cập vận hành và khả năng quan sát (observability) của hạ tầng AWS đã được chuẩn bị trong các lab mạng và tính toán trước đó.

Thay vì triển khai thêm một workload mới, tuần này tập trung vào việc quản lý một EC2 instance hiện có thông qua các phương thức truy cập do AWS quản lý, thu thập dữ liệu đo đạc mạng VPC, và chuyển đổi lưu lượng mạng bị từ chối thành tín hiệu giám sát CloudWatch.

Quá trình triển khai bao gồm AWS Systems Manager Session Manager, EC2 Instance Connect Endpoint, VPC Flow Logs, CloudWatch Logs, một bộ lọc metric tùy chỉnh (custom metric filter), và một CloudWatch alarm.

## Bối cảnh Dự án

Dự án Hybrid IDS/AI SOC yêu cầu nhiều hơn một backend instance đang chạy. Hạ tầng cũng phải hỗ trợ quyền truy cập quản trị có kiểm soát, khắc phục sự cố mạng, dữ liệu đo đạc bảo mật (security telemetry), và giám sát vận hành.

Kết nối SSH trực tiếp qua địa chỉ IPv4 công khai rất hữu ích trong quá trình triển khai ban đầu, nhưng không nên là con đường truy cập quản trị duy nhất.

Do đó, Tuần 4 đã đánh giá hai mô hình truy cập do AWS quản lý:

- Session Manager cho phép quản trị qua trình duyệt thông qua Systems Manager.
- EC2 Instance Connect Endpoint cung cấp quyền truy cập SSH tạm thời qua đường dẫn IPv4 riêng (private IPv4 path).

VPC cũng được cấu hình để xuất bản Flow Logs lên CloudWatch Logs. Các bản ghi lưu lượng bị từ chối (REJECT) được chuyển đổi thành một metric tùy chỉnh và được đánh giá bởi một CloudWatch alarm.

## Tổng quan Kiến trúc Vận hành

Công việc Tuần 4 có thể được hiểu theo hai luồng vận hành liên kết với nhau: truy cập quản trị có kiểm soát và khả năng quan sát mạng.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-operations-architecture.svg" alt="Tổng quan kiến trúc vận hành Tuần 4">
  <figcaption>Hình 1 - Luồng truy cập vận hành và quan sát mạng (network-observability) của Tuần 4 cho môi trường EC2 SOC MVP hiện tại.</figcaption>
</figure>

Đường dẫn truy cập xác nhận cách quản trị viên có thể tiếp cận backend instance thông qua các cơ chế do AWS quản lý. Đường dẫn quan sát xác nhận cách siêu dữ liệu lưu lượng VPC có thể trở thành một metric CloudWatch và sau đó là một alarm.

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Lab vận hành và ghi chú gốc | 11/05/2026 | Xem xét khái niệm EIC Endpoint, Session Manager, VPC Flow Logs, Reachability Analyzer và CloudWatch | Phạm vi vận hành và khả năng quan sát được xác định cho Tuần 4 | Một số tài nguyên và ảnh chụp màn hình gốc không đủ để làm bằng chứng cuối cùng | Tái xác nhận các bằng chứng còn thiếu nếu tài nguyên còn có thể sử dụng |
| Tái xác nhận bằng chứng và triển khai bổ sung | 12/05/2026 | Gắn IAM role SSM, xác minh Session Manager, xác minh truy cập EIC qua private-IP, cấu hình chuyển phát Flow Logs, đặt thời gian lưu trữ log, tạo REJECT metric filter, tạo CloudWatch alarm, và thu thập bằng chứng an toàn để công khai | Các kiểm soát truy cập an toàn và quan sát mạng đã được xác minh cho môi trường SOC MVP đang hoạt động | Bằng chứng Reachability Analyzer chưa được thu thập và vẫn được hoãn lại | Hoàn thiện ảnh chụp màn hình, xem xét dọn dẹp, và tiếp tục với S3/CloudFront |

## Tóm tắt Triển khai Kỹ thuật

Tuần 4 mở rộng hạ tầng SOC MVP hiện có bằng hai phương thức truy cập quản trị do AWS quản lý. Backend EC2 instance được đăng ký với Systems Manager và truy cập thông qua Session Manager, trong khi EC2 Instance Connect Endpoint cung cấp đường dẫn SSH thông qua địa chỉ IPv4 riêng của instance.

Dữ liệu đo đạc mạng được kích hoạt thông qua VPC Flow Logs và CloudWatch Logs. Các bản ghi REJECT được chuyển đổi thành metric tùy chỉnh `SOC/Network` `RejectedConnections`, từ đó kích hoạt CloudWatch alarm `SOC-Rejected-Connections`.

## Truy cập Vận hành An toàn

Session Manager và EC2 Instance Connect Endpoint giải quyết các vấn đề truy cập liên quan nhưng khác nhau.

| Phương thức truy cập | Mục đích | Phụ thuộc chính |
| --- | --- | --- |
| Session Manager | Quản trị qua trình duyệt thông qua Systems Manager | IAM role của EC2, SSM Agent, kết nối Systems Manager |
| EC2 Instance Connect Endpoint | Đường dẫn SSH tạm thời qua tuyến đường VPC riêng | EIC Endpoint, đường dẫn Security Group, SSH daemon, quyền IAM |

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e03-session-manager-managed-node.png" alt="Systems Manager managed node">
  <figcaption>Hình 2 - Backend EC2 instance đã đăng ký thành công như một managed node trực tuyến trên Systems Manager.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Node xuất hiện trực tuyến với thông tin Linux và metadata EC2.  
**Phạm vi:** Xác nhận đăng ký managed-node, không phải triển khai ứng dụng.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e04-session-manager-start-session.png" alt="Session Manager shell">
  <figcaption>Hình 3 - Một phiên shell Session Manager qua trình duyệt cung cấp quyền truy cập quản trị mà không cần phiên SSH cục bộ trực tiếp.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Shell được mở với tư cách `ssm-user` và trả về kết quả tên máy chủ và thư mục làm việc.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e02-eic-private-ip-connect-success.png" alt="Kết nối EIC qua private IPv4 thành công">
  <figcaption>Hình 4 - Backend EC2 instance hiện tại được truy cập qua địa chỉ IPv4 riêng bằng EC2 Instance Connect Endpoint.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Shell Ubuntu được mở qua đích đến private IPv4 và trả về kết quả user, hostname và interface.  
**Phạm vi:** Xác nhận đường dẫn truy cập qua địa chỉ IPv4 riêng của instance. Điều này không ngụ ý rằng EC2 instance hiện có đã được chuyển sang private subnet.

## VPC Flow Logs và CloudWatch Logs

VPC Flow Logs được sử dụng như dữ liệu đo đạc mạng đám mây. Chúng không thu thập nội dung gói tin như Wireshark hay Zeek, nhưng cung cấp siêu dữ liệu hữu ích như nguồn, đích, giao thức, cổng, số gói tin, số byte, hành động (action) và trạng thái log.

Trong lab này, Flow Logs được chuyển đến CloudWatch Logs. Thời gian lưu trữ của log group được giới hạn ở bảy ngày để tránh lưu trữ log mạng lab vô thời hạn.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e07-flow-log-record.png" alt="Bản ghi Flow Log trong CloudWatch">
  <figcaption>Hình 5 - CloudWatch Logs đã nhận các bản ghi VPC Flow Log, bao gồm cả lưu lượng được chấp nhận (accepted) và bị từ chối (rejected).</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Các bản ghi đã nhận bao gồm các trường trạng thái `ACCEPT`, `REJECT` và `OK`.  
**Che giấu thông tin:** Account ID và các cột IP công khai không cần thiết đã được ẩn đi trong khi vẫn giữ nguyên các trường action và status.

## Bộ lọc Metric và CloudWatch Alarm

Chuỗi giám sát được xây dựng trong Tuần 4 là:

```text
Lưu lượng VPC
-> Bản ghi Flow Log
-> CloudWatch Logs
-> metric filter
-> custom metric
-> CloudWatch alarm
```

Bộ lọc metric khớp với các bản ghi Flow Log có `action="REJECT"` và phát ra một giá trị vào namespace `SOC/Network`.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e09-flowlogs-rejected-metric-filter.png" alt="Bộ lọc metric RejectedConnections">
  <figcaption>Hình 6 - Bộ lọc metric RejectedConnections chuyển đổi các bản ghi VPC Flow Log REJECT thành một metric CloudWatch tùy chỉnh.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Bộ lọc phát ra giá trị `1` vào metric `SOC/Network` `RejectedConnections` khi có các sự kiện log `REJECT` mới.

<figure class="worklog-evidence">
  <img src="/images/worklog/week4/w4-e08-cloudwatch-rejected-connections-alarm.png" alt="CloudWatch alarm cho rejected connections">
  <figcaption>Hình 7 - Alarm SOC-Rejected-Connections đã chuyển sang trạng thái ALARM sau khi custom metric đạt đến ngưỡng đã cấu hình.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Alarm đánh giá `RejectedConnections >= 1` trong một điểm dữ liệu trong vòng một phút.  
**Phạm vi:** Không có thông báo SNS nào được thêm vào vì lab này tập trung vào quy trình giám sát, không phải cảnh báo qua email.

## Ma trận Xác minh

| Kiểm tra | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| --- | --- | --- | --- |
| Đăng ký EC2 với Systems Manager | Instance xuất hiện trực tuyến | Managed node xuất hiện trực tuyến | Đạt |
| Truy cập Session Manager | Shell trình duyệt mở | Phiên khởi động với tư cách `ssm-user` | Đạt |
| Truy cập EIC qua private-IP | SSH shell mở qua đường dẫn private IPv4 | Ubuntu shell mở và trả về kết quả host/network | Đạt |
| Chuyển phát Flow Log | Bản ghi đến CloudWatch Logs | Các bản ghi `ACCEPT` và `REJECT` được nhận | Đạt |
| Chuyển đổi REJECT metric | Sự kiện `REJECT` mới phát ra một giá trị metric | Bộ lọc metric `RejectedConnections` đã được tạo | Đạt |
| Đánh giá Alarm | Metric lớn hơn hoặc bằng `1` thay đổi trạng thái alarm | Alarm chuyển sang trạng thái `ALARM` | Đạt |
| Reachability Analyzer | Kết quả phân tích đường dẫn được ghi lại | Không thực hiện với bằng chứng công khai | Hoãn lại |

Hai ảnh chụp màn hình đã bị loại khỏi trang chính vì chúng là bằng chứng yếu hơn: ảnh chi tiết EIC endpoint vẫn hiển thị trạng thái pending, và ảnh cài đặt Flow Log hiển thị lỗi xác thực service-role. Trang cuối cùng sử dụng bằng chứng theo dõi mạnh hơn thay thế.

## Quyết định Bảo mật, Chi phí và Dọn dẹp

| Quyết định | Lý do |
| --- | --- |
| Sử dụng IAM role cho EC2 | Tránh lưu trữ thông tin xác thực AWS dài hạn trên instance |
| Xác minh Session Manager | Giảm sự phụ thuộc vào SSH công khai và các file `.pem` cục bộ |
| Sử dụng Security Group reference cho EIC | Giới hạn TCP 22 theo đường dẫn truy cập endpoint |
| Tạm thời giữ quy tắc `/32` của quản trị viên | Tránh bị khóa ra ngoài trong khi khắc phục sự cố truy cập EIC |
| Đặt thời gian lưu trữ log là bảy ngày | Giới hạn lưu trữ CloudWatch Logs không cần thiết |
| Không sử dụng SNS action | Tuần 4 chỉ xác minh quy trình giám sát |
| Hoãn Reachability Analyzer | Không có bằng chứng công khai đáng tin cậy được thu thập |
| Xem xét tài nguyên tạm thời sau khi thu thập bằng chứng | Tránh giữ lại các endpoint, filter, alarm và log không sử dụng |

| Tài nguyên hoặc kiểm soát | Trạng thái dọn dẹp |
| --- | --- |
| EIC Endpoint | Giữ lại cho việc tái xác nhận bằng chứng và xem xét tiếp theo |
| VPC Flow Log | Giữ lại trong khi bằng chứng quan sát Tuần 4 được xem xét |
| Metric filter | Giữ lại cùng với log group Flow Logs trong quá trình xem xét bằng chứng |
| CloudWatch alarm | Giữ lại cho xác minh Tuần 4; không có SNS action được cấu hình |
| Log group | Giữ lại với thời gian lưu trữ bảy ngày |
| SSM role | Giữ lại cho các hoạt động vận hành backend |

## Vấn đề và Khắc phục

| Vấn đề | Giải pháp |
| --- | --- |
| Kết nối EIC qua trình duyệt ban đầu thất bại | Xác minh trạng thái endpoint và thêm một quy tắc SSH inbound riêng cho backend có nguồn từ Security Group của EIC Endpoint |
| Quy tắc SSH hiện có không thể chuyển đổi từ CIDR sang Security Group reference | Tạm thời giữ quy tắc `/32` của quản trị viên và thêm một quy tắc nguồn Security Group mới |
| Flow Logs yêu cầu CloudWatch publishing role | Tạo hoặc chọn một service role dành riêng cho VPC Flow Logs |
| CloudWatch log group đã tồn tại | Sử dụng lại group hiện có và cấu hình thời gian lưu trữ bảy ngày |
| Biểu đồ custom metric ban đầu hiển thị trống | Mở metric namespace, chọn `RejectedConnections`, và tạo lưu lượng REJECT mới |
| Alarm mới ban đầu thiếu đủ datapoints | Xem xét cách xử lý dữ liệu thiếu và tạo sự kiện REJECT mới sau khi lọc |
| Bằng chứng Reachability Analyzer không có sẵn | Đánh dấu tính năng này là hoãn lại thay vì khẳng định xác minh thành công |

Việc xác minh trong tương lai nên phân tích đường dẫn TCP/22 giữa EIC Endpoint ENI và backend ENI.

## Bài học rút ra

Vận hành hạ tầng an toàn đòi hỏi các kiểm soát riêng biệt cho định danh (identity), truy cập mạng, dữ liệu đo đạc (telemetry) và cảnh báo.

Session Manager và EC2 Instance Connect Endpoint giải quyết các vấn đề truy cập liên quan nhưng khác nhau. Session Manager cung cấp quyền truy cập quản trị được quản lý thông qua Systems Manager, trong khi EIC Endpoint cung cấp đường dẫn SSH tạm thời qua VPC bằng địa chỉ IPv4 riêng.

Truy cập private IPv4 cũng không phải là cùng khẳng định với việc đặt instance trong private subnet. Kiểm tra EIC Tuần 4 chứng minh đường dẫn truy cập qua địa chỉ IPv4 riêng, trong khi việc đặt subnet là một phần của thiết kế VPC và tính toán rộng hơn.

Các quy tắc Security Group cần được mô hình hóa đúng cách. Một quy tắc IPv4 CIDR hiện có không thể chỉ đơn giản được chuyển đổi thành một Security Group reference. Phải tạo một quy tắc mới cho đường dẫn từ endpoint đến instance.

VPC Flow Logs chỉ hữu ích sau khi xác minh việc chuyển phát thành công. Việc tạo tài nguyên Flow Log hay log group không đồng nghĩa với việc xác nhận rằng các bản ghi thực tế đang được nhận.

Chuỗi giám sát cũng chứa nhiều giai đoạn riêng biệt. Mỗi giai đoạn phải được xác minh độc lập để tránh nhầm lẫn giữa một tài nguyên đã cấu hình với một tín hiệu giám sát đang hoạt động.

## Kết quả đạt được trong tuần

Vào cuối Tuần 4, môi trường EC2 SOC MVP hiện tại đã có hai phương thức truy cập vận hành do AWS quản lý và một quy trình giám sát bảo mật mạng cơ bản.

EC2 instance được đăng ký với Systems Manager và truy cập thông qua Session Manager. EC2 Instance Connect Endpoint cũng được sử dụng để kết nối đến instance qua địa chỉ IPv4 riêng của nó.

VPC Flow Logs đã chuyển các bản ghi mạng đến CloudWatch Logs. Lưu lượng bị từ chối được chuyển đổi thành metric tùy chỉnh `RejectedConnections`, và một CloudWatch alarm đã được tạo và kích hoạt để đánh giá tín hiệu đó.

Reachability Analyzer vẫn là tùy chọn và không được trình bày như một xác minh hoàn chỉnh vì bằng chứng tương ứng chưa được thu thập.

## Kế hoạch tuần tiếp theo

Tuần 5 sẽ chuyển từ vận hành tính toán sang lưu trữ và phân phối nội dung tĩnh. Trọng tâm sẽ là:

- Thiết kế lưu trữ Amazon S3
- Cấu hình bucket frontend riêng tư (private frontend bucket)
- S3 versioning và lifecycle settings
- Amazon CloudFront distribution
- Origin Access Control
- Giới hạn truy cập S3 trực tiếp
- Phân phối frontend tĩnh an toàn thông qua CloudFront
