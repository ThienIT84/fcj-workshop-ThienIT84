---
title: "Tuần 7 - Phân đoạn Mạng IDS Cục bộ & Xác minh Zeek Telemetry"
date: 2026-05-31
weight: 7
chapter: false
draft: false
pre: " <b> 1.7. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Trọng tâm chính | Đường cơ sở lab IDS cục bộ |
| Mục tiêu chính | Xác minh phân đoạn mạng và Zeek telemetry trước khi tạo dataset |
| Firewall/router | pfSense |
| Máy tấn công | Kali Linux trong DMZ / OPT1 |
| Máy nạn nhân | Ubuntu Victim trong LAN |
| Máy telemetry | Zeek sensor trên LAN |
| Bằng chứng chính | Quy tắc/log pfSense, kiểm thử Kali-đến-Victim, khả năng quan sát gói tin Zeek, `conn.log`, `http.log`, xuất dataset |
| Kibana visualization | Hoãn lại |
| Trạng thái trong tuần | Đã triển khai từ ghi chú lab cục bộ; xác minh lại bằng chứng hoàn tất |

## Lưu ý Xác minh lại Bằng chứng

Lab IDS cục bộ đã được xây dựng trước khi trang worklog Tuần 7 này được chuẩn bị. Tuần 7 không khẳng định rằng pfSense, Kali, Ubuntu Victim và Zeek được cài đặt từ đầu trong phiên xác minh lại bằng chứng.

Thay vào đó, worklog Tuần 7 tập trung vào việc xác minh lab hiện có:

```text
xem xét topology
-> xác nhận các interface pfSense
-> thắt chặt quy tắc firewall OPT1
-> xác minh quyền truy cập Kali-đến-Victim
-> xem xét hành vi default-deny
-> xác nhận dịch vụ của Victim
-> xác nhận khả năng quan sát gói tin của Zeek
-> kiểm tra conn.log và http.log
-> xuất dataset có cấu trúc
```

Bằng chứng được thu thập lại vào ngày `01/07/2026` từ lab cục bộ đang hoạt động. Các ảnh chụp màn hình là bản sao công khai đã được cắt gọn, lưu trong:

```text
static/images/worklog/week7/
```

Kibana không được khôi phục trong phiên xác minh lại bằng chứng này, vì vậy Kibana visualization vẫn được hoãn lại.

## Mục tiêu

Mục tiêu của Tuần 7 là chứng minh rằng lab IDS cục bộ có thể tạo ra traffic mạng có kiểm soát và chuyển đổi nó thành Zeek telemetry.

Điều này quan trọng vì dự án Hybrid IDS/AI SOC không nên chỉ phụ thuộc vào các dataset công khai. Lab cục bộ cung cấp nguồn traffic có kiểm soát, vai trò rõ ràng, IP xác định và nhãn có thể giải thích để tạo dataset và huấn luyện mô hình AI sau này.

## Bối cảnh Dự án

Tuần 1–6 đã thiết lập nền tảng AWS, lưu trữ, cơ sở dữ liệu, phân phối nội dung tĩnh và các khái niệm giám sát vận hành. Tuần 7 chuyển trọng tâm sang nguồn telemetry.

Lab cục bộ cung cấp các tín hiệu bảo mật thô có thể sau này được:

- capture bởi Zeek,
- xuất dưới dạng `conn.log` và `http.log`,
- phân tích thành CSV dataset,
- tải lên S3 như artifact,
- phân tích bởi các thành phần phát hiện AI,
- tổng hợp bởi SOC backend.

Do đó, Tuần 7 là cầu nối giữa việc học hạ tầng và pipeline dữ liệu bảo mật.

## Tổng quan Kiến trúc IDS Cục bộ

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e01-local-ids-topology.svg" alt="Topology lab IDS cục bộ">
  <figcaption>Hình 1 - Topology lab IDS cục bộ sử dụng để xác minh phân đoạn mạng và Zeek telemetry trong Tuần 7.</figcaption>
</figure>

Topology đã xác minh là:

| Phân đoạn | Thành phần | Địa chỉ IP |
| --- | --- | --- |
| VMware NAT / VMnet8 | pfSense WAN | `192.168.44.133/24` |
| DMZ / OPT1 | pfSense OPT1 | `10.10.10.1/24` |
| DMZ / OPT1 | Kali attacker | `10.10.10.10/24` |
| LAN | pfSense LAN | `192.168.1.1/24` |
| LAN | Ubuntu Victim | `192.168.1.10/24` |
| LAN | Zeek sensor | `192.168.1.20/24` |

Đường traffic dự kiến là:

```text
Kali 10.10.10.10
-> pfSense OPT1
-> pfSense LAN
-> Victim 192.168.1.10
-> Zeek telemetry trên LAN
```

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày tháng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Xem xét topology lab IDS cục bộ | 31/05/2026 | Xem xét topology VM pfSense, Kali, Victim và Zeek, ghi lại mô hình địa chỉ DMZ/LAN | Mô hình phân đoạn lab được định nghĩa xung quanh Kali trong OPT1 và Victim/Zeek trong LAN | Giữ IP lab riêng tư hiển thị vì chúng giải thích bằng chứng | Xác minh các interface pfSense |
| Truy cập pfSense và xem xét quy tắc | 02/06/2026 | Khôi phục quyền truy cập WebGUI pfSense, xác nhận interface, xác định rằng quy tắc troubleshooting `OPT1 net -> Any` quá rộng, và thay thế bằng quy tắc host-to-host | Kali bị giới hạn chỉ đến mục tiêu Victim thay vì truy cập LAN không hạn chế | WebGUI dùng HTTP port 80 trong lab; ảnh chụp màn hình công khai đã được cắt để bỏ thanh cảnh báo trình duyệt | Xác minh các đường traffic được phép và bị chặn |
| Xác minh kết nối Victim và Kali | 03/06/2026 | Xác nhận IP/gateway của Victim, trạng thái Apache và SSH, route của Kali đến Victim, ICMP, HTTP response và kết quả Nmap service | Kali có thể đến Victim HTTP và SSH qua pfSense | Dịch vụ Victim là mục tiêu lab, không phải exposure production | Xác minh khả năng quan sát của Zeek |
| Xác minh khả năng quan sát và log Zeek | 05/06/2026 | Dùng `tcpdump` trên Zeek VM trong khi Kali tạo traffic, kiểm tra runtime Zeek, kiểm tra `conn.log`, kiểm tra `http.log`, và tương quan bản ghi bằng Zeek UID | Zeek thấy traffic Kali-đến-Victim và tạo ra metadata kết nối/ứng dụng | Chạy Zeek chưa đủ; khả năng quan sát gói tin và nội dung log phải được kiểm tra | Xuất telemetry thành dataset có cấu trúc |
| Xác minh lại bằng chứng và xuất dataset | 07/06/2026 | Thu thập lại ảnh chụp màn hình Tuần 7, xem xét bằng chứng pass/default-deny của pfSense, xuất CSV dataset có cấu trúc, kiểm tra shape, cột và phân phối nhãn của dataset | Bằng chứng Tuần 7 được hợp nhất thành ảnh chụp màn hình an toàn để công khai | Kibana không được khôi phục để chụp ảnh màn hình và vẫn được hoãn lại | Dùng Tuần 8 cho các traffic campaign và event diary |

## Tóm tắt Triển khai Kỹ thuật

Tuần 7 đã xác minh một mạng IDS cục bộ nơi Kali được cô lập trong phân đoạn kiểu DMZ và các máy Victim/Zeek được đặt trong phân đoạn LAN.

pfSense định tuyến traffic giữa hai phân đoạn và thực thi chính sách truy cập. Quy tắc OPT1 cuối cùng chỉ cho phép:

```text
Source:      10.10.10.10
Destination: 192.168.1.10
Protocol:    IPv4 any
```

Điều này vẫn cho phép traffic lab có kiểm soát như ICMP, HTTP, SSH và Nmap testing nhắm vào Victim. Quy tắc troubleshooting rộng hơn trước đó đã bị vô hiệu hóa.

Zeek sau đó được xác minh từ phía telemetry. Khả năng quan sát gói tin được xác nhận bằng `tcpdump`, và log Zeek được kiểm tra qua `conn.log` và `http.log`. Xuất dataset có cấu trúc cũng được kiểm tra bằng Pandas.

## Phân đoạn Mạng và Quy tắc pfSense

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e02-pfsense-interface-assignments.png" alt="Gán interface pfSense">
  <figcaption>Hình 2 - Gán interface pfSense xác nhận địa chỉ WAN, LAN, OPT1 và OPT2.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** WAN, LAN và OPT1 có sẵn, với LAN trên `192.168.1.1/24` và OPT1 trên `10.10.10.1/24`.  
**Phạm vi:** OPT2 hiển thị nhưng không là một phần của đường telemetry Tuần 7.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e03-pfsense-opt1-firewall-rule.png" alt="Quy tắc firewall OPT1 pfSense">
  <figcaption>Hình 3 - Quy tắc OPT1 cho phép Kali `10.10.10.10` chỉ đến Victim `192.168.1.10`.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Quy tắc host-to-host đang hoạt động, trong khi quy tắc troubleshooting rộng hơn trước đó là `OPT1 net -> Any` đã bị vô hiệu hóa.  
**Giá trị bảo mật:** Kali vẫn hữu ích cho việc tạo traffic có kiểm soát mà không nhận được quyền truy cập LAN không hạn chế.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e03a-pfsense-kali-victim-pass-log.png" alt="Log pass pfSense cho Kali đến Victim">
  <figcaption>Hình 4 - Log firewall pfSense hiển thị traffic Kali-đến-Victim khớp với quy tắc cho phép.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Traffic ICMP, TCP/80 và TCP/22 từ `10.10.10.10` đến `192.168.1.10` khớp với quy tắc `Allow Kali attacker to Victim lab target`.  
**Tại sao quan trọng:** Một quy tắc firewall nên được xác minh bằng log traffic, không chỉ qua màn hình cấu hình của nó.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e03b-pfsense-kali-zeek-block-log.png" alt="Log firewall default-deny pfSense">
  <figcaption>Hình 5 - Một ví dụ log default-deny pfSense được chụp trong quá trình xác minh lại firewall.</figcaption>
</figure>

**Kết quả:** Bằng chứng hỗ trợ  
**Quan sát chính:** Ảnh chụp màn hình minh họa ghi log firewall default-deny, nhưng nó không lưu rõ ràng bộ đôi đầy đủ Kali-đến-Zeek trong bản crop an toàn để công khai.  
**Lưu ý xuất bản:** Trước khi chuyển trang này sang công khai, nên thêm ảnh chụp màn hình mạnh hơn nếu báo cáo cần bằng chứng trực quan về việc `10.10.10.10 -> 192.168.1.20` bị chặn.

## Xác minh Victim và Kali

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e04-victim-network-services.png" alt="Mạng và dịch vụ Ubuntu Victim">
  <figcaption>Hình 6 - Ubuntu Victim có địa chỉ LAN mong đợi và các dịch vụ HTTP/SSH được mở cho lab.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Victim có `192.168.1.10/24`, default gateway `192.168.1.1`, Apache đang hoạt động, SSH đang hoạt động, TCP/80 đang lắng nghe và TCP/22 đang lắng nghe.  
**Phạm vi:** Những dịch vụ này là các mục tiêu lab có chủ ý để tạo telemetry.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e05-kali-to-victim-validation.png" alt="Xác minh Kali đến Victim">
  <figcaption>Hình 7 - Kali đến Victim thành công qua ICMP, HTTP và phát hiện dịch vụ.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Kali dùng `10.10.10.10`, định tuyến đến `192.168.1.10` qua pfSense, nhận HTTP `200 OK`, và phát hiện SSH/HTTP bằng Nmap.  
**Giá trị dự án:** Điều này chứng minh đường traffic có kiểm soát cần thiết trước khi thu thập IDS log.

## Khả năng Quan sát Traffic của Zeek

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e06-zeek-live-traffic-visibility.png" alt="Khả năng quan sát traffic trực tiếp của Zeek qua tcpdump">
  <figcaption>Hình 8 - Zeek VM quan sát traffic trực tiếp Kali-đến-Victim trên `ens33`.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** `tcpdump` hiển thị gói tin theo cả hai chiều giữa `10.10.10.10` và `192.168.1.10`, bao gồm traffic HTTP và SSH.  
**Tại sao quan trọng:** Cài đặt Zeek không chứng minh khả năng quan sát; interface sensor thực sự phải thấy traffic.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e07-zeek-runtime-status.png" alt="Trạng thái runtime Zeek">
  <figcaption>Hình 9 - Cài đặt Zeek và context runtime được kiểm tra trên telemetry VM.</figcaption>
</figure>

**Kết quả:** Bằng chứng hỗ trợ  
**Quan sát chính:** Phiên bản Zeek và các network interface của máy chủ hiển thị, và context process/runtime đã được xem xét.  
**Phạm vi:** Trạng thái runtime hỗ trợ bằng chứng gói tin/log, nhưng không đủ bởi chính nó.

## Xác minh Log Zeek và Tương quan UID

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e08-zeek-conn-log.png" alt="Xác minh conn log Zeek">
  <figcaption>Hình 10 - `conn.log` chứa metadata cấp kết nối cho traffic HTTP Kali-đến-Victim.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Bản ghi `conn.log` bao gồm `10.10.10.10` là origin, `192.168.1.10` là responder, TCP/80, service `http`, bytes, packets, connection state và Zeek UID.  
**Giá trị dự án:** `conn.log` cung cấp các tính năng cấp mạng cốt lõi để tạo dataset sau này.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e09-zeek-http-uid-correlation.png" alt="Tương quan UID http log Zeek">
  <figcaption>Hình 11 - `http.log` tiết lộ metadata HTTP và chia sẻ Zeek UID với bản ghi kết nối.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** HTTP method, host, URI, trường status và UID có mặt cho các request từ `10.10.10.10` đến `192.168.1.10`.  
**Giá trị tương quan:** UID liên kết các bản ghi cấp kết nối trong `conn.log` với các bản ghi cấp ứng dụng trong `http.log`.

## Xác minh Xuất Dataset

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e10-normal-http-session.png" alt="Tạo phiên HTTP bình thường">
  <figcaption>Hình 12 - Các HTTP request lặp lại tạo ra traffic baseline bình thường để Zeek capture.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Các HTTP request lặp lại trả về phản hồi thành công và tạo ra traffic baseline để so sánh với hoạt động đáng ngờ.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e11-controlled-port-scan.png" alt="Traffic port scan có kiểm soát">
  <figcaption>Hình 13 - Port scan có kiểm soát tạo ra traffic lab đáng ngờ nhưng có giới hạn nhắm vào Victim.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Scan được giới hạn ở source đã biết, destination đã biết và dải port xác định.  
**Phạm vi:** Điều này chứng minh việc tạo và ghi lại traffic đáng ngờ, không phải độ chính xác phát hiện cấp production.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e12-zeek-dataset-export.png" alt="Xác minh xuất dataset Zeek">
  <figcaption>Hình 14 - Telemetry từ Zeek được xuất thành CSV dataset có cấu trúc và kiểm tra bằng Pandas.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Shape dataset, danh sách cột và phân phối nhãn đã được kiểm tra.  
**Phạm vi:** Điều này chỉ xác minh việc xuất dataset. Missing values, duplicates, class balance, chất lượng nhãn, leakage và AI readiness vẫn là công việc của Tuần 9.

## Ma trận Xác minh

| Xác minh | Kết quả thực tế | Trạng thái |
| --- | --- | --- |
| Các interface pfSense | WAN, LAN và địa chỉ OPT1 được xác minh | Đạt |
| Phân đoạn Kali | Kali được đặt trong `10.10.10.0/24` | Đạt |
| Phân đoạn Victim | Victim được đặt trong `192.168.1.0/24` | Đạt |
| Quy tắc OPT1 hạn chế | Kali chỉ được phép đến Victim trong thiết kế quy tắc cuối cùng | Đạt |
| Kiểm tra firewall tích cực | Traffic Kali-đến-Victim được ghi lại là pass | Đạt |
| Kiểm tra firewall tiêu cực | Hành vi block được ghi lại trong notes; khuyến nghị screenshot tuple mạnh hơn | Xem xét trước khi xuất bản |
| HTTP/SSH của Victim | TCP/80 và TCP/22 đang hoạt động | Đạt |
| Truy cập Kali-đến-Victim | Xác minh ping, HTTP và Nmap thành công | Đạt |
| Khả năng quan sát gói tin Zeek | Sensor thấy traffic Kali-đến-Victim | Đạt |
| `conn.log` | Các trường source, destination, service và UID chính xác có mặt | Đạt |
| `http.log` | HTTP method, URI và trường UID có mặt | Đạt |
| Tương quan UID | Các lớp kết nối và HTTP có thể được liên kết bằng UID | Đạt |
| Traffic bình thường | Traffic HTTP baseline được ghi lại | Đạt |
| Traffic đáng ngờ | Traffic scan có kiểm soát được tạo ra và ghi lại | Đạt |
| Xuất dataset | CSV có cấu trúc được kiểm tra bằng Pandas | Đạt |
| Kibana visualization | ELK/Kibana không được khôi phục trong quá trình xác minh lại | Hoãn lại |
| Sẵn sàng huấn luyện mô hình | Kiểm tra chất lượng dữ liệu chưa hoàn thành trong Tuần 7 | Hoãn lại |

## Quyết định Bảo mật

| Quyết định | Lý do |
| --- | --- |
| Đặt Kali trong phân đoạn kiểu DMZ | Cô lập vai trò attacker khỏi giám sát và hạ tầng LAN |
| Định tuyến traffic Kali qua pfSense | Buộc traffic qua điểm chính sách có thể quan sát |
| Giới hạn Kali chỉ đến một IP Victim | Ngăn quyền truy cập lateral không hạn chế bên trong lab |
| Vô hiệu hóa quy tắc troubleshooting OPT1 rộng | Tránh để `OPT1 net -> Any` là trạng thái cuối cùng |
| Bật logging trên quy tắc cho phép | Chứng minh quy tắc khớp với traffic thực tế |
| Giữ Zeek/ELK tách biệt khỏi Kali | Bảo vệ hệ thống giám sát khỏi quyền truy cập trực tiếp của attacker |
| Giữ IP lab riêng tư hiển thị | Chúng cần thiết để giải thích topology và bằng chứng |
| Giữ log thô và PCAP riêng tư | Chúng có thể chứa dữ liệu traffic chi tiết ngoài báo cáo công khai |
| Hoãn claim Kibana | Tránh khẳng định bằng chứng visualization chưa được capture mới |

## Vấn đề và Khắc phục

| Vấn đề | Nguyên nhân gốc | Giải pháp |
| --- | --- | --- |
| pfSense HTTPS bị timeout | WebGUI đang lắng nghe trên HTTP port `80`, không phải HTTPS `443` | Kiểm tra trạng thái service đang lắng nghe và dùng URL lab chính xác |
| Trình duyệt hiển thị cảnh báo quản lý không bảo mật | WebGUI lab dùng HTTP trong quá trình xác minh lại | Cắt ảnh chụp màn hình công khai và ghi lại đây là giới hạn quản lý chỉ dành cho lab |
| Cảnh báo mật khẩu admin mặc định xuất hiện trong ảnh chụp màn hình pfSense | pfSense hiển thị banner cảnh báo lab | Cắt bằng chứng công khai để banner không được xuất bản |
| Quy tắc OPT1 quá rộng | Quy tắc tạm thời `OPT1 net -> Any` được dùng để troubleshooting | Thêm quy tắc cho phép host-to-host và vô hiệu hóa quy tắc rộng |
| Zeek VM có nhiều IP LAN tại một thời điểm | Interface có cả địa chỉ dự định và địa chỉ thêm | Dùng kiểm tra routing/source-IP và ghi lại rủi ro cấu hình |
| Cần chứng minh khả năng quan sát của sensor | Cài đặt Zeek một mình không chứng minh gói tin có thể quan sát | Dùng `tcpdump` trong khi tạo traffic Kali-đến-Victim |
| Bằng chứng Kibana không có sẵn | ELK stack không được khôi phục để chụp ảnh màn hình | Đánh dấu Kibana visualization là hoãn lại |
| Screenshot block tiêu cực yếu | Bản crop công khai không rõ ràng hiển thị bộ đôi Kali-đến-Zeek | Giữ như bằng chứng default-deny hỗ trợ và khuyến nghị bằng chứng mạnh hơn trước khi xuất bản |

## Bài học rút ra

Bằng chứng phân đoạn mạng nên hiển thị cả cấu hình và hành vi. Màn hình quy tắc firewall hữu ích, nhưng log firewall chứng minh traffic có thực sự khớp với quy tắc dự kiến hay không.

Ping thành công không có nghĩa là giao diện web có thể truy cập qua HTTPS. Troubleshooting nên kiểm tra ICMP, trạng thái TCP port và service listener thực tế.

Zeek phải được xác minh ở ba cấp: khả năng quan sát interface, trạng thái process/runtime và nội dung log. Bằng chứng mạnh nhất không phải là `zeek --version`; mà là traffic được sensor thấy và được biểu diễn trong `conn.log` và `http.log`.

Tương quan Zeek UID quan trọng cho công việc dataset vì nó liên kết metadata cấp kết nối với bản ghi cấp ứng dụng.

Xuất dataset không giống với dataset sẵn sàng huấn luyện mô hình. Tuần 7 chứng minh rằng xuất có cấu trúc là có thể, trong khi chất lượng dữ liệu và xác minh nhãn nên được xử lý sau.

## Kết quả đạt được trong tuần

Vào cuối Tuần 7, lab IDS cục bộ có một đường cơ sở phân đoạn mạng và telemetry đã được xác minh.

Kali được cô lập trong phân đoạn OPT1/DMZ và được phép đến máy Victim qua quy tắc pfSense có kiểm soát. Các dịch vụ HTTP và SSH của Victim đang hoạt động để tạo traffic lab. Zeek quan sát traffic trực tiếp Kali-đến-Victim, tạo ra metadata kết nối và HTTP, và xuất telemetry thành dataset có cấu trúc.

Kibana visualization và xác minh sẵn sàng huấn luyện mô hình đầy đủ được hoãn lại một cách có chủ ý để tránh các khẳng định không có cơ sở.

## Kế hoạch tuần tiếp theo

Tuần 8 sẽ chuyển từ xác minh lab sang các traffic campaign và quản lý event diary.

Trọng tâm sẽ là:

- Các traffic campaign bình thường,
- Các traffic campaign tấn công có kiểm soát,
- Event diary và theo dõi timestamp,
- Thu thập `conn.log` và `http.log` từ Zeek,
- Bằng chứng Suricata hoặc IDS bổ sung nếu có,
- Tách biệt rõ ràng giữa raw traffic, parsed log và labeled events.
