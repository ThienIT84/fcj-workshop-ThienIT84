---
title: "Tuần 5 - Tầng dữ liệu RDS PostgreSQL"
date: 2026-05-17
weight: 5
chapter: false
draft: false
pre: " <b> 1.5. </b> "
---

## Tổng quan tuần qua

| Hạng mục | Kết quả |
| --- | --- |
| Dịch vụ chính | Amazon RDS |
| Database engine | PostgreSQL |
| Các dịch vụ hỗ trợ | VPC, DB Subnet Group, EC2, Security Groups |
| Mục tiêu chính | Xây dựng tầng cơ sở dữ liệu quản trị (managed database) riêng tư cho dữ liệu cảnh báo SOC |
| Region chính | `ap-southeast-1` - Châu Á Thái Bình Dương (Singapore) |
| Mô hình mạng | Client EC2 công khai kết nối đến RDS PostgreSQL riêng tư |
| Mô hình truy cập DB | Security Group của RDS chỉ cho phép cổng PostgreSQL `5432` từ Security Group của EC2 |
| Kết quả dự án | Bảng `alerts` với các trường mức độ nghiêm trọng (severity), thông điệp (message), và điểm bất thường (anomaly score) |
| Trạng thái tuần | Đã triển khai với các bằng chứng chọn lọc công khai |

## Ghi chú nguồn bằng chứng

Bằng chứng thực hành RDS Tuần 5 được tổng hợp vào ngày `28/05/2026` tại:

```text
FCAJ-Internship/01_AWS-Labs/Lab005_RDS/
```

Nỗ lực triển khai kéo dài hơn một buổi thực hành ngắn. Các ảnh chụp màn hình được lưu lại thể hiện kết quả kiểm thử cuối cùng, trong khi nhật ký hàng ngày dưới đây chia nhỏ quá trình thực hành RDS qua nhiều buổi làm việc thực tế.

Các tài nguyên AWS gốc đã được xóa trước khi trang nhật ký này được chuẩn bị, do đó trang này tái sử dụng các ghi chú lab và ảnh chụp màn hình đã lưu để làm bằng chứng. Các ảnh chụp gốc được lưu trữ riêng tư, trong khi trang công khai sử dụng các hình ảnh đã lược bỏ thông tin nhạy cảm tại:

```text
static/images/worklog/week5/
```

## Mục tiêu

Mục tiêu của Tuần 5 là thiết kế và xác thực tầng lưu trữ dữ liệu quan hệ được quản trị cho dự án Hybrid IDS/AI SOC.

Log thô, tập dữ liệu và các model artifact phù hợp hơn với lưu trữ đối tượng (object storage), nhưng các cảnh báo cuối cùng, điểm bất thường và bản ghi dashboard cần truy vấn có cấu trúc. Do đó, Amazon RDS PostgreSQL đã được lựa chọn làm dịch vụ cơ sở dữ liệu quản trị cho tầng dữ liệu có cấu trúc này.

## Bối cảnh dự án

Dự án SOC cần một nơi để lưu trữ lâu dài các phát hiện bảo mật sau khi dữ liệu telemetry thô được xử lý bởi backend hoặc AI worker.

Trong thiết kế dự án, các sự kiện từ Zeek, Suricata hoặc VPC Flow Logs có thể được chuyển đổi thành các bản ghi cảnh báo (alert). Các bản ghi này sau đó được lưu trong PostgreSQL và truy vấn bởi dashboard thông qua API backend.

Tuần 5 tập trung vào mô hình truy cập này:

```text
Laptop
-> SSH vào client EC2 công khai
-> psql qua cổng PostgreSQL 5432 có mã hóa SSL
-> RDS PostgreSQL riêng tư (private)
-> Bảng alerts lưu các bản ghi dạng SOC
```

Instance EC2 công khai được sử dụng làm database client tạm thời phục vụ cho bài lab học tập. Trong kiến trúc SOC mục tiêu, backend ứng dụng sẽ sử dụng cùng mô hình truy cập dựa trên Security Group từ một subnet riêng tư (private subnet), mà không yêu cầu bản thân cơ sở dữ liệu phải công khai ra internet.

| Phiên bản triển khai | Đường truyền truy cập |
| --- | --- |
| Phòng lab thực hành | Laptop → client EC2 công khai → RDS riêng tư |
| Kiến trúc mục tiêu | Backend ứng dụng (subnet riêng tư) → RDS riêng tư |

Thông tin xác thực cơ sở dữ liệu chỉ được sử dụng cho kết nối lab và không được đưa vào báo cáo Hugo, ảnh chụp màn hình, mã nguồn hoặc kho lưu trữ Git.

## Tổng quan kiến trúc RDS

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-rds-architecture.svg" alt="RDS PostgreSQL architecture overview">
  <figcaption>Hình 1 - Kiến trúc RDS PostgreSQL Tuần 5 để lưu trữ cảnh báo SOC và điểm số bất thường.</figcaption>
</figure>

Cơ sở dữ liệu được đặt phía sau ranh giới VPC và được truy cập thông qua một đường truyền Security Group được kiểm soát. Client EC2 hoạt động như máy chủ ứng dụng thử nghiệm, trong khi RDS vẫn ở chế độ riêng tư và chỉ có thể truy cập qua endpoint cơ sở dữ liệu.

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày | Công việc hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- |
| Lập kế hoạch mạng và kiến trúc RDS | 17/05/2026 | Xem xét các case sử dụng RDS PostgreSQL, lên kế hoạch cho VPC chuyên dụng, phân bổ subnet công khai/riêng tư, DB subnet group và đường truyền truy cập client EC2 | Kiến trúc lab RDS được định nghĩa xoay quanh việc đặt database trong mạng riêng tư | RDS không được để công khai; client EC2 có thể đặt ở public subnet để phục vụ truy cập lab có kiểm soát | Xây dựng các điều kiện tiên quyết về mạng |
| Chuẩn bị VPC, subnet và Security Group | 18/05/2026 | Tạo VPC cho lab RDS, chuẩn bị các subnet public/private, tạo Security Group cho EC2 và RDS, cấu hình quyền truy cập PostgreSQL `5432` từ EC2-SG sang RDS-SG | Baseline mạng và kiểm soát truy cập đã sẵn sàng cho RDS | Việc tham chiếu Security Group chỉ hoạt động chính xác khi các tài nguyên nằm trong cùng một VPC | Tạo DB subnet group và deploy RDS instance |
| Tạo DB subnet group và triển khai RDS PostgreSQL | 20/05/2026 | Tạo DB subnet group chứa các private subnet trên hai Availability Zone và triển khai một instance RDS PostgreSQL tối ưu chi phí | RDS PostgreSQL đã sẵn sàng hoạt động với tính năng public access bị tắt | Tắt các tính năng chi phí cao của production không cần thiết cho bài lab | Chuẩn bị client EC2 và kiểm thử truy cập database |
| Thiết lập client EC2 và khắc phục sự cố kết nối | 22/05/2026 | SSHed vào EC2, cài đặt các công cụ client PostgreSQL, kiểm thử lệnh `psql`, khắc phục lỗi timeout/xác thực và thêm cấu hình mã hóa `sslmode=require` | EC2 kết nối thành công đến RDS PostgreSQL qua SSL | Lỗi timeout do cấu hình mạng/Security Group; lỗi không mã hóa yêu cầu sử dụng chế độ SSL | Tạo schema cảnh báo SOC và chạy các câu lệnh truy vấn |
| Tạo schema cảnh báo SOC, xác thực truy vấn và viết tài liệu | 23/05/2026 | Tạo bảng `alerts`, chèn dữ liệu sự kiện SOC mẫu, truy vấn bản ghi theo mức độ nghiêm trọng và điểm số bất thường, chụp bằng chứng cuối cùng và ghi chép bài học | RDS lưu trữ và phản hồi dữ liệu cảnh báo dạng SOC thành công | Việc tích hợp FastAPI cần các bằng chứng riêng biệt trước khi được ghi nhận | Tiếp tục với S3/CloudFront để phân phối static web và lưu trữ artifact |

## Tóm tắt triển khai kỹ thuật

Tuần 5 đã xây dựng một mạng lab RDS chuyên dụng sử dụng dải CIDR `10.20.0.0/16` với các subnet public và private trải trên hai Availability Zone.

Một DB Subnet Group được tạo từ các subnet riêng tư trong `ap-southeast-1a` and `ap-southeast-1b`, cung cấp cho RDS khả năng triển khai subnet riêng tư hợp lệ trên hai AZ. Một instance EC2 công khai đóng vai trò là client để kết nối SSH và chạy các lệnh kiểm thử database.

RDS PostgreSQL được triển khai dưới dạng một instance lab nhỏ và tắt tính năng public access. Security Group của database chỉ cho phép lưu lượng truy cập PostgreSQL trên cổng `5432` đi từ Security Group của client EC2.

Sau khi cài đặt PostgreSQL client trên EC2, kết nối tới RDS được kiểm tra bằng cách sử dụng `sslmode=require`. Bài lab sau đó đã tạo một bảng `alerts` và chèn các bản ghi SOC mẫu như các sự kiện brute force SSH và quét cổng (port scan).

## Mạng và vị trí đặt cơ sở dữ liệu

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e01-db-subnet-group-private-subnets.png" alt="DB subnet group private subnets">
  <figcaption>Hình 2 - DB subnet group lựa chọn các subnet riêng tư trên hai Availability Zone.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Subnet group sử dụng các subnet riêng tư trong `ap-southeast-1a` và `ap-southeast-1b` với các dải CIDR `10.20.128.0/20` và `10.20.144.0/20`.  
**Giá trị bảo mật:** Vị trí đặt RDS được kiểm soát độc lập với quyền truy cập.

> [!NOTE]
> DB Subnet Group bao gồm các subnet riêng tư trên hai Availability Zone, nhưng bản thân cơ sở dữ liệu lab chỉ sử dụng triển khai instance đơn (single-instance). Cấu hình subnet này không nên được hiểu là một triển khai cơ sở dữ liệu Multi-AZ đã được xác thực.

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e02-rds-security-group-rule.png" alt="RDS Security Group PostgreSQL rule">
  <figcaption>Hình 3 - Security Group của RDS cho phép PostgreSQL 5432 từ Security Group của client EC2.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Luật inbound PostgreSQL sử dụng tham chiếu Security Group của EC2 thay vì mở rộng `0.0.0.0/0`.  
**Phạm vi:** Điều này chứng minh đường truyền truy cập database mong muốn, không phải là một chính sách mạng cấp production.

## Instance RDS PostgreSQL

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e03-rds-instance-available.png" alt="RDS PostgreSQL available">
  <figcaption>Hình 4 - Instance RDS PostgreSQL ở trạng thái sẵn sàng (available) với tính năng truy cập internet công khai bị tắt.</figcaption>
</figure>

**Kết quả:** Đã xác thực  
**Quan sát chính:** Cơ sở dữ liệu sử dụng engine PostgreSQL, class instance là `db.t3.micro`, và tắt tính năng public accessibility (`Publicly accessible: No`).  
**Lưu ý chi phí:** Bài lab sử dụng một class instance nhỏ và tránh các tính năng có tính sẵn sàng cao (high-availability) hoặc giám sát nâng cao không cần thiết.

## Kết nối Client EC2 và Cơ sở dữ liệu

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e04-postgresql-client-install.png" alt="PostgreSQL client installed on EC2">
  <figcaption>Hình 5 - Các công cụ client PostgreSQL được cài đặt trên instance client EC2.</figcaption>
</figure>

**Kết quả:** Đã xác thực  
**Quan sát chính:** Máy chủ EC2 đã cài đặt thành công `postgresql15` và xác nhận phiên bản `psql` là `15.16`.

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e05-rds-ssl-connection-success.png" alt="RDS SSL connection success">
  <figcaption>Hình 6 - Client EC2 kết nối tới RDS PostgreSQL sử dụng SSL và truy vấn database/user hiện tại.</figcaption>
</figure>

**Kết quả:** Đã xác thực  
**Quan sát chính:** Phiên kết nối `psql` đã truy cập thành công vào `fcaj_db`, sử dụng SSL, và trả về thông tin database cùng user hiện tại.  
**Giá trị khắc phục sự cố:** Các nỗ lực xác thực/kết nối trước đó hiển thị rõ trên màn hình, cho thấy lệnh thành công cuối cùng có được sau khi sửa chuỗi kết nối và chế độ SSL.

## Mô hình dữ liệu cảnh báo SOC

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e06-alerts-table-created.png" alt="Alerts table created in PostgreSQL">
  <figcaption>Hình 7 - Bảng `alerts` được tạo với các trường kiểu sự kiện, mức độ nghiêm trọng, địa chỉ IP, thông điệp, điểm bất thường và timestamp.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Schema của bảng được thiết kế phù hợp với nhu cầu hiển thị của dashboard SOC thay vì một cơ sở dữ liệu demo chung chung.

<figure class="worklog-evidence">
  <img src="/images/worklog/week5/w5-e07-alert-query-results.png" alt="Alert query results in PostgreSQL">
  <figcaption>Hình 8 - Các bản ghi cảnh báo mẫu được chèn và truy vấn theo mức độ nghiêm trọng và điểm số bất thường.</figcaption>
</figure>

**Kết quả:** Đã xác thực  
**Quan sát chính:** Kết quả truy vấn hiển thị rõ ràng các bản ghi `SSH brute force` và `Port scan` đi kèm các giá trị về mức độ nghiêm trọng và điểm bất thường.

## Ma trận xác thực

| Kiểm thử | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| --- | --- | --- | --- |
| Vị trí subnet DB | RDS subnet group sử dụng các subnet riêng tư trên hai AZ | Các subnet riêng tư được chọn nằm trong `ap-southeast-1a` và `ap-southeast-1b` | Pass |
| Kiểm soát truy cập RDS | PostgreSQL `5432` chỉ được phép đi từ EC2-SG, không từ internet công khai | RDS-SG sử dụng nguồn là Security Group của EC2 | Pass |
| Trạng thái sẵn sàng của RDS | Instance PostgreSQL chuyển sang trạng thái available | `fcaj-postgres-db` hiển thị sẵn sàng kết nối | Pass |
| Truy cập database công khai | Cơ sở dữ liệu không thể truy cập công khai | Cấu hình hiển thị `Publicly accessible: No` | Pass |
| Database client trên EC2 | `psql` được cài đặt trên EC2 | Đã xác nhận phiên bản `psql (PostgreSQL) 15.16` | Pass |
| Truy cập database qua SSL | EC2 kết nối tới RDS thông qua mã hóa SSL | `psql` kết nối thành công qua SSL và trả về thông tin `fcaj_db` | Pass |
| Lưu trữ dữ liệu SOC | Bảng alert lưu trữ các bản ghi có thể truy vấn được | Bảng `alerts` chèn và trả về chính xác hai bản ghi mẫu | Pass |
| Tích hợp FastAPI | API backend đọc các bản ghi trực tiếp từ RDS | Không được hỗ trợ bởi các bằng chứng ảnh chụp lab Tuần 5 | Deferred |

## Vấn đề và Khắc phục sự cố

| Vấn đề | Hướng giải quyết |
| --- | --- |
| Kết nối RDS ban đầu bị timeout | Kiểm tra xem EC2 và RDS Security Group có nằm trong cùng một VPC hay không và RDS-SG đã cho phép PostgreSQL `5432` từ EC2-SG hay chưa |
| EC2-SG và RDS-SG cấu hình sai lệch | Tạo lại hoặc sửa đổi cấu hình Security Group để cả hai tài nguyên đều tuân theo đúng mô hình VPC lab RDS |
| Xác thực mật khẩu thất bại | Kiểm tra lại thông tin username/password của cơ sở dữ liệu và thử lại kết nối `psql` |
| Kết nối trước đó không yêu cầu mã hóa SSL | Thêm tham số `sslmode=require` vào chuỗi kết nối PostgreSQL và xác nhận phiên kết nối đã được mã hóa |
| RDS có thể phát sinh chi phí lớn nếu duy trì chạy | Xóa các tài nguyên lab sau khi thu thập đủ bằng chứng và duy trì trang báo cáo dựa trên bằng chứng đã lưu thay vì duy trì tài nguyên hoạt động |

## Quyết định An ninh, Chi phí và Dọn dẹp

| Quyết định | Lý do |
| --- | --- |
| Đặt RDS trong các subnet riêng tư | Cơ sở dữ liệu không nên tiếp xúc trực tiếp với môi trường internet công cộng |
| Sử dụng DB Subnet Group | Kiểm soát chính xác vị trí mà cơ sở dữ liệu được quản lý có thể triển khai |
| Sử dụng EC2-SG làm nguồn cho RDS-SG | Giới hạn quyền truy cập database chỉ từ máy chủ ứng dụng hoặc client được chỉ định |
| Sử dụng `sslmode=require` cho kết nối `psql` | Xác thực việc mã hóa dữ liệu truyền tải (transit encryption) trong suốt bài lab |
| Sử dụng class instance lab nhỏ `db.t3.micro` | Tiết kiệm tối đa chi phí chạy thử nghiệm |
| Triển khai Single-AZ và bỏ qua RDS Proxy | Các tính năng này cần thiết cho production nhưng không cần thiết cho việc xác thực bài lab ngắn hạn |
| Tắt tính năng deletion protection cho lab | Cho phép dọn dẹp sạch tài nguyên sau khi thu thập đủ bằng chứng |
| Lưu trữ log thô ở nơi khác | RDS dùng để lưu trữ các bản ghi alert có cấu trúc, không dùng cho các file log thô dung lượng lớn |
| Giữ thông tin đăng nhập database ngoài ảnh chụp và source code | Ngăn chặn rò rỉ thông tin đăng nhập vào báo cáo công khai hoặc kho mã nguồn Git |
| Sử dụng thông tin đăng nhập thủ công tạm thời | Việc tích hợp Secrets Manager nằm ngoài phạm vi của Tuần 5 |
| Trì hoãn quản lý secret ứng dụng sang giai đoạn migrate | Luồng tích hợp Secrets Manager hoặc Parameter Store sẽ được đưa vào trong đợt chạy sprint tiếp theo |

## Quyết định phân bổ lưu trữ dữ liệu

Không phải tất cả dữ liệu SOC đều thuộc về cơ sở dữ liệu quan hệ. Lab Tuần 5 sử dụng PostgreSQL cho các bản ghi alert cuối cùng, trong khi dữ liệu telemetry thô và model artifact phù hợp hơn ở kho lưu trữ đối tượng.

| Loại dữ liệu | Bộ nhớ khuyến nghị | Lý do |
| --- | --- | --- |
| Log Zeek/Suricata thô | Amazon S3 | Các artifact dạng log dung lượng lớn, ghi nối tiếp phù hợp cho lưu trữ dài hạn |
| Bản lưu trữ VPC Flow Log | Amazon S3 | Bản ghi mạng lưu lượng lớn phù hợp để lưu trữ dưới dạng đối tượng |
| Model artifact | Amazon S3 | Các artifact triển khai dựa trên đối tượng |
| Cảnh báo cuối cùng (Final alerts) | PostgreSQL | Cần truy vấn có cấu trúc, lọc, sắp xếp và lưu trữ hiển thị trên dashboard |
| Điểm số và trạng thái alert | PostgreSQL | Backend và dashboard cần thực hiện các câu lệnh truy vấn quan hệ |
| Dữ liệu xử lý tạm thời | Bộ nhớ cục bộ / bộ nhớ tạm | Dữ liệu trung gian không nhất thiết cần lưu trữ quan hệ bền vững |

## Bài học kinh nghiệm

DB Subnet Group và Security Group giải quyết các bài toán khác nhau. DB Subnet Group quyết định nơi RDS được triển khai, trong khi Security Group quyết định ai được phép kết nối tới RDS đó.

Lỗi kết nối bị timeout thường chỉ ra các vấn đề về định tuyến mạng hoặc Security Group, chứ không phải do mật khẩu. Lỗi xác thực mật khẩu (authentication error) và lỗi SSL chỉ xuất hiện sau khi đường truyền mạng đã tiếp cận được cơ sở dữ liệu thành công.

Ứng dụng nên sử dụng endpoint của RDS thay vì địa chỉ IP DB cố định. Endpoint là địa chỉ kết nối ổn định hướng ứng dụng ngay cả khi AWS thay đổi máy chủ cơ sở dữ liệu vật lý bên dưới.

Đối với dự án SOC này, PostgreSQL là lựa chọn tối ưu cho các bản ghi alert cuối cùng vì dashboard yêu cầu bộ lọc có cấu trúc, sắp xếp, gom nhóm và truy vấn theo mức độ nghiêm trọng.

Tham số `sslmode=require` giúp mã hóa phiên kết nối cơ sở dữ liệu. Bước thiết lập nâng cao trên production sẽ yêu cầu xác thực thêm chuỗi chứng chỉ (certificate chain) và hostname của RDS bằng tham số `sslmode=verify-full`, thay vì chỉ dựa vào mã hóa truyền tải đơn thuần.

## Kết quả tuần qua

Vào cuối Tuần 5, mô hình tầng dữ liệu RDS PostgreSQL riêng tư đã được triển khai thành công và xác thực thông qua quyền truy cập từ client EC2.

Bài lab đã chứng minh RDS có khả năng lưu trữ các bản ghi cảnh báo SOC có cấu trúc trong khi vẫn duy trì ranh giới bảo mật riêng tư. Client EC2 kết nối qua SSL, tạo bảng `alerts`, chèn các phát hiện mẫu và truy vấn thành công dữ liệu theo mức độ nghiêm trọng và điểm số bất thường.

Phần tích hợp FastAPI với RDS vẫn là hạng mục cần tiếp tục triển khai tiếp theo.

## Kế hoạch tuần tiếp theo

Tuần 6 sẽ chuyển từ lưu trữ dữ liệu có cấu trúc sang lưu trữ đối tượng và phân phối tĩnh. Trọng tâm sẽ là:

- Thiết kế S3 bucket
- Lưu trữ đối tượng thô riêng tư cho các tập dữ liệu SOC
- Hosting static website trên S3 cho các tệp giao diện
- Thiết lập Bucket Policy chọn lọc thay vì mở công khai toàn bộ bucket
- Cấu hình phân phối Amazon CloudFront sử dụng endpoint website của S3 làm origin
- Xác thực đường dẫn CloudFront URL
- Thiết lập quy tắc S3 Versioning và Lifecycle
- Phân tách rõ ràng giữa các đối tượng thô riêng tư và tài sản website công khai
