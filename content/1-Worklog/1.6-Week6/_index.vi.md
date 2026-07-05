---
title: "Tuần 6 - Website Tĩnh S3, CloudFront & Vòng đời Đối tượng"
date: 2026-05-29
weight: 6
chapter: false
draft: true
pre: " <b> 1.6. </b> "
---

## Tổng quan trong tuần

| Hạng mục | Kết quả |
| --- | --- |
| Dịch vụ trọng tâm | Amazon S3 |
| Dịch vụ hỗ trợ | Amazon CloudFront, S3 Versioning, S3 Lifecycle, Bucket Policy |
| Mục tiêu chính | Xây dựng mô hình lưu trữ đối tượng và phân phối nội dung tĩnh cho các artifacts của dự án FCAJ |
| Region chính | `ap-southeast-1` - Asia Pacific (Singapore) |
| Mô hình lưu trữ | Đối tượng thô riêng tư (private raw objects) cộng với các file website tĩnh được chọn lọc công khai |
| Mô hình phân phối | CloudFront distribution sử dụng S3 static website endpoint làm origin |
| Trọng tâm dữ liệu | Tập dữ liệu Zeek-style thô, file website tĩnh, các đối tượng demo versioning |
| Trạng thái trong tuần | Đã triển khai với bằng chứng công khai được chọn lọc; ước tính thời gian cần chủ nhân xác nhận |

## Lưu ý về Nguồn Bằng chứng

Bằng chứng Tuần 6 được chuẩn bị từ các ghi chú lab S3 hiện có trong:

```text
FCAJ-Internship/01_AWS-Labs/S3/
```

Các ảnh chụp màn hình đã lưu được chụp vào khoảng ngày `29/05/2026`. Các tài nguyên S3 và CloudFront gốc có thể đã bị xóa, vì vậy trang này sử dụng các ghi chú lab và ảnh chụp màn hình đã lưu làm bằng chứng.

Các ảnh chụp màn hình gốc được lưu trữ riêng tư. Báo cáo công khai sử dụng các bản sao đã được cắt gọn dưới:

```text
static/images/worklog/week6/
```

Lab này sử dụng S3 static website endpoint kết hợp bucket policy có chọn lọc và CloudFront. Lab không triển khai phân phối CloudFront qua private S3 origin access control. Việc hardening private-bucket CloudFront delivery vẫn là hạng mục cải thiện trong tương lai.

## Mục tiêu

Mục tiêu của Tuần 6 là tìm hiểu cách Amazon S3 có thể hỗ trợ dự án Hybrid IDS/AI SOC như một lớp lưu trữ đối tượng và cách CloudFront có thể phục vụ nội dung tĩnh được chọn lọc thông qua một CDN endpoint.

Lab tập trung vào bốn tính năng liên kết với nhau:

- Lưu trữ các đối tượng liên quan đến bảo mật như tập dữ liệu Zeek-style thô.
- Mặc định giữ riêng tư các đối tượng thô nhạy cảm.
- Chỉ công khai các file website tĩnh được chọn lọc.
- Sử dụng versioning và lifecycle rules để quản lý lịch sử đối tượng và dọn dẹp.

## Bối cảnh Dự án

Dự án SOC tạo ra nhiều loại dữ liệu khác nhau. Không phải tất cả đều phù hợp lưu trong cùng một dịch vụ lưu trữ.

Amazon RDS hữu ích cho các bản ghi alert có cấu trúc và truy vấn dashboard, như đã được kiểm tra trong Tuần 5. Amazon S3 phù hợp hơn cho dữ liệu lớn dạng đối tượng như log, CSV dataset, model file, báo cáo tĩnh và frontend build artifact.

Do đó, Tuần 6 đã kiểm thử một mô hình lưu trữ và phân phối có thể hỗ trợ cả lab học tập lẫn SOC dashboard trong tương lai:

```text
Tập dữ liệu Zeek-style thô   -> S3 object riêng tư trong raw/zeek/
File website tĩnh             -> S3 objects công khai được chọn lọc
CloudFront                    -> HTTPS viewer endpoint trước S3 website endpoint
Versioning                    -> rollback đối tượng và bảo vệ khỏi ghi đè vô tình
Lifecycle rule                -> dọn dẹp các phiên bản cũ và delete markers
```

## Tổng quan Kiến trúc S3 và CloudFront

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-s3-cloudfront-architecture.svg" alt="Tổng quan kiến trúc S3 và CloudFront Tuần 6">
  <figcaption>Hình 1 - Kiến trúc lưu trữ và phân phối nội dung tĩnh Tuần 6 sử dụng S3, selective website access, CloudFront, versioning và lifecycle cleanup.</figcaption>
</figure>

Kiến trúc này chủ động tách biệt các project artifacts riêng tư khỏi các file website tĩnh. Dữ liệu bảo mật thô được tải lên S3 nhưng không được công khai. Chỉ `index.html` và `error.html` được phép truy cập công khai thông qua lab bucket policy, cho phép S3 website endpoint và CloudFront distribution phục vụ website tĩnh.

## Nhật ký công việc hàng ngày

| Hoạt động | Ngày | Thời lượng | Công việc đã hoàn thành | Kết quả | Vấn đề / Quyết định | Bước tiếp theo |
| --- | --- | --- | --- | --- | --- | --- |
| Chuẩn bị đường cơ sở S3 bucket | 29/05/2026 | Ước tính 3 giờ | Tạo S3 lab bucket trong `ap-southeast-1`, xem xét đặt tên bucket, object ownership, Block Public Access, trạng thái mặc định versioning, tags và mã hóa SSE-S3 mặc định | Đường cơ sở S3 bucket riêng tư đã được chuẩn bị cho project artifacts | Giữ bucket riêng tư theo mặc định trước khi công khai bất kỳ file website nào | Tải lên các đối tượng theo phong cách dự án |
| Tải lên đối tượng thô và kiểm tra truy cập riêng tư | 30/05/2026 | Ước tính 3.5 giờ | Tạo prefix logic `raw/zeek/`, tải lên `conn_dataset.csv`, và mở URL đối tượng từ trình duyệt | Đối tượng dataset thô tồn tại trên S3 và truy cập trực tiếp qua trình duyệt trả về `AccessDenied` | Log bảo mật và dataset không nên là public web asset | Chỉ bật website hosting cho các file được chọn lọc |
| Hosting website tĩnh và bucket policy | 31/05/2026 | Ước tính 4 giờ | Tạo `index.html` và `error.html`, bật S3 static website hosting, tắt Block Public Access cấp bucket cho lab, và thêm selective bucket policy | S3 website endpoint phục vụ trang FCAJ tĩnh thành công | Policy chỉ cho phép truy cập file website, không phải toàn bộ bucket prefix | Thêm CloudFront trước website endpoint |
| Xác minh phân phối CloudFront | 01/06/2026 | Ước tính 4 giờ | Tạo CloudFront distribution, cấu hình S3 static website endpoint làm origin, sử dụng chứng chỉ CloudFront mặc định, và kiểm thử CloudFront domain | Website truy cập được qua CloudFront URL | Origin dùng HTTP vì S3 website endpoints không hỗ trợ HTTPS đến origin | Thêm versioning và lifecycle management |
| Versioning, lifecycle và tổ chức đối tượng | 02/06/2026 | Ước tính 4 giờ | Bật S3 Versioning, tải lên nhiều phiên bản của đối tượng demo, xem xét hành vi delete-marker, tạo lifecycle rule cho `versioning-demo/`, và ghi lại hành vi dọn dẹp | Lịch sử đối tượng và lifecycle cleanup đã được xác minh cho lab prefix | Lifecycle rules không thực thi ngay lập tức; chúng chạy bất đồng bộ dựa trên tuổi đời và điều kiện | Dùng Tuần 7 để chuyển từ cloud storage sang kiến trúc IDS telemetry cục bộ |

## Tóm tắt Triển khai Kỹ thuật

Lab Tuần 6 tạo S3 bucket tên `fcaj-s3-lab-thienit-20260529` trong `ap-southeast-1`.

Bucket ban đầu ở chế độ riêng tư. Object Ownership dùng ACLs disabled, mã hóa mặc định dùng SSE-S3, và Block Public Access được bật trong giai đoạn tải lên đối tượng thô ban đầu.

Tập dữ liệu Zeek-style được tải lên trong:

```text
raw/zeek/conn_dataset.csv
```

URL đối tượng trực tiếp trả về `AccessDenied`, đây là kết quả mong đợi cho dữ liệu bảo mật thô.

Đối với phần website tĩnh, bucket host `index.html` và `error.html`. Bucket policy được cố tình giới hạn phạm vi chỉ cho hai file website đó thay vì sử dụng policy `bucket/*` rộng. CloudFront sau đó được cấu hình với S3 static website endpoint làm origin và phục vụ website qua CloudFront distribution domain.

Lab cũng bật S3 Versioning và tạo lifecycle rule tên `cleanup-versioning-demo` cho prefix `versioning-demo/`.

## Đường cơ sở Lưu trữ S3

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e01-s3-bucket-created.png" alt="S3 bucket đã được tạo">
  <figcaption>Hình 2 - S3 lab bucket được tạo trong `ap-southeast-1` và dùng làm đường cơ sở lưu trữ đối tượng.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Tên bucket và Region hiển thị rõ ràng, và bucket đã được chuẩn bị trước khi cấu hình bất kỳ quyền truy cập website nào.  
**Giá trị dự án:** S3 cung cấp lớp lưu trữ đối tượng bền vững cho log thô, dataset, model artifact và report file.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e02-raw-zeek-object-uploaded.png" alt="Đối tượng Zeek thô được tải lên S3">
  <figcaption>Hình 3 - Đối tượng tập dữ liệu Zeek-style được tải lên dưới prefix logic `raw/zeek/`.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** `raw/zeek/conn_dataset.csv` được xử lý như một object key trong S3, không phải folder hệ thống file thực sự.  
**Giá trị dự án:** Dự án SOC có thể lưu trữ raw telemetry và các dataset đã tạo ra trong object storage trước khi xử lý về sau.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e03-private-object-access-denied.png" alt="Truy cập đối tượng S3 riêng tư bị từ chối">
  <figcaption>Hình 4 - Truy cập trực tiếp qua trình duyệt vào đối tượng dataset thô trả về phản hồi AccessDenied.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Đối tượng thô không thể đọc công khai thông qua URL đối tượng trực tiếp của nó.  
**Giá trị bảo mật:** Log bảo mật thô và dataset nên được giữ riêng tư trừ khi một đường dẫn ứng dụng có kiểm soát cho phép truy cập một cách tường minh.

## Website Tĩnh và Bucket Policy

S3 static website hosting được bật với:

```text
Index document: index.html
Error document: error.html
```

Lab chủ ý chỉ công khai các file website. Điều này khác với một bucket policy rộng làm lộ mọi đối tượng trong bucket.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e05-selective-bucket-policy.png" alt="Selective S3 bucket policy cho file website">
  <figcaption>Hình 5 - Bucket policy cấp quyền đọc công khai chỉ cho `index.html` và `error.html`.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Policy sử dụng `s3:GetObject` cho hai tài nguyên cụ thể thay vì `arn:aws:s3:::bucket-name/*`.  
**Phạm vi:** Đây là mô hình website tĩnh ở giai đoạn lab. Nó an toàn hơn so với việc công khai toàn bộ bucket, nhưng không tương đương với phân phối CloudFront từ private-bucket origin.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e04-static-website-success.png" alt="S3 static website endpoint thành công">
  <figcaption>Hình 6 - S3 static website endpoint phục vụ FCAJ static website thành công.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Trình duyệt hiển thị trang `FCAJ - S3 Static Website` qua S3 website endpoint.  
**Lưu ý bảo mật:** S3 website endpoint dùng HTTP. CloudFront được thêm vào tiếp theo để cung cấp HTTPS viewer endpoint.

## Phân phối CloudFront

CloudFront được cấu hình trước S3 static website endpoint.

Điểm phân biệt quan trọng là:

| Tùy chọn origin | Trạng thái Tuần 6 | Ghi chú |
| --- | --- | --- |
| S3 static website endpoint | Đã sử dụng | Cần thiết cho hành vi S3 website hosting như index và error document |
| S3 REST endpoint với private origin access | Chưa triển khai | Hạng mục hardening production trong tương lai |

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e06-cloudfront-origin-website-endpoint.png" alt="CloudFront origin dùng S3 website endpoint">
  <figcaption>Hình 7 - CloudFront được cấu hình với S3 website endpoint làm origin domain.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Origin domain sử dụng `s3-website-ap-southeast-1.amazonaws.com`, không phải S3 REST endpoint.  
**Phạm vi:** Phù hợp với bằng chứng lab và tránh cường điệu hóa mô hình bảo mật.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e07-cloudfront-distribution-detail.png" alt="Chi tiết CloudFront distribution">
  <figcaption>Hình 8 - CloudFront distribution được tạo cho FCAJ S3 static website lab.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Distribution có CloudFront domain name và `index.html` là default root object.  
**Lưu ý chi phí:** Lab sử dụng CloudFront domain mặc định và không thêm custom domain, chứng chỉ ACM, WAF hay các tính năng edge-security trả phí.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e08-cloudfront-url-success.png" alt="CloudFront URL phục vụ website tĩnh thành công">
  <figcaption>Hình 9 - CloudFront URL phục vụ FCAJ static website thành công.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Cùng một trang tĩnh được hiển thị qua CloudFront distribution domain.  
**Giá trị dự án:** Mô hình này có thể sau này phục vụ SOC frontend hoặc các public internship report asset.

## Versioning và Quản lý Vòng đời

S3 Versioning được sử dụng để hiểu lịch sử đối tượng, ghi đè và delete markers.

Lab sử dụng một đối tượng nhỏ trong:

```text
versioning-demo/version-demo.txt
```

Điều này tránh phải tải lên lặp lại file dataset lớn hơn chỉ để kiểm tra hành vi versioning.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e09-version-history.png" alt="Lịch sử phiên bản đối tượng S3">
  <figcaption>Hình 10 - S3 Versioning hiển thị nhiều phiên bản của cùng một object key.</figcaption>
</figure>

**Kết quả:** Đã xác minh  
**Quan sát chính:** Tải lên cùng một object key sau khi bật versioning tạo ra nhiều phiên bản thay vì chỉ đơn giản ghi đè đối tượng trước.  
**Giá trị dự án:** Versioning có thể bảo vệ project report, model artifact và các file cấu hình nhỏ khỏi bị ghi đè vô tình.

<figure class="worklog-evidence">
  <img src="/images/worklog/week6/w6-e10-lifecycle-rule-enabled.png" alt="S3 lifecycle rule đã bật">
  <figcaption>Hình 11 - Lifecycle rule `cleanup-versioning-demo` được bật để kiểm thử version-cleanup.</figcaption>
</figure>

**Kết quả:** Đã triển khai  
**Quan sát chính:** Lifecycle rule được lọc và bật cho hành vi cleanup thay vì áp dụng mù quáng cho toàn bộ bucket.  
**Giá trị chi phí:** Versioning có thể làm tăng mức sử dụng storage, vì vậy lifecycle policy là kiểm soát bổ sung quan trọng để ngăn các phiên bản cũ và delete markers tích lũy.

## Ma trận Xác minh

| Kiểm tra | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| --- | --- | --- | --- |
| Tạo S3 bucket | Bucket tồn tại trong `ap-southeast-1` | Bucket `fcaj-s3-lab-thienit-20260529` đã được tạo | Đạt |
| Tải lên đối tượng thô | Đối tượng dataset được lưu dưới prefix theo phong cách dự án | `raw/zeek/conn_dataset.csv` đã được tải lên | Đạt |
| Tính riêng tư của đối tượng thô | Truy cập trực tiếp qua trình duyệt không nên làm lộ dataset | Object URL trả về AccessDenied | Đạt |
| File website tĩnh | `index.html` và `error.html` có thể dùng làm file website | Website endpoint hiển thị trang FCAJ tĩnh | Đạt |
| Phạm vi bucket policy | Website policy không nên làm lộ toàn bộ bucket | Policy chỉ liệt kê `index.html` và `error.html` | Đạt |
| CloudFront origin | Distribution trỏ đến S3 website endpoint | Origin domain sử dụng website endpoint | Đạt |
| Phân phối CloudFront | CloudFront URL hiển thị website tĩnh | CloudFront domain trả về trang FCAJ | Đạt |
| Hành vi Versioning | Tải lại cùng key tạo ra các phiên bản | Danh sách version hiển thị nhiều object version | Đạt |
| Quản lý Lifecycle | Cleanup rule tồn tại cho các phiên bản cũ/delete markers | Rule `cleanup-versioning-demo` đã được bật | Đạt |
| S3 origin riêng tư qua CloudFront | Bucket hoàn toàn riêng tư trong khi chỉ CloudFront phục vụ nội dung | Chưa triển khai trong lab này | Cải thiện trong tương lai |

## Quyết định Bảo mật, Chi phí và Dọn dẹp

| Quyết định | Lý do |
| --- | --- |
| Giữ riêng tư các đối tượng dataset thô | Log bảo mật thô và dataset đã tạo ra không nên là public web asset |
| Sử dụng selective bucket policy | Chỉ `index.html` và `error.html` cần quyền đọc công khai cho lab website |
| Tránh policy `bucket/*` rộng | Ngăn vô tình làm lộ các đối tượng trong `raw/zeek/` |
| Dùng CloudFront cho viewer delivery | Cung cấp URL công khai rõ ràng hơn và HTTPS viewer access cho website tĩnh |
| Sử dụng S3 website endpoint một cách trung thực | Bằng chứng hỗ trợ website endpoint origin, không phải private REST-origin hardening |
| Hoãn private-origin hardening | Cần bằng chứng mới và mô hình CloudFront/S3 access khác |
| Dùng Versioning trên đối tượng demo nhỏ | Tránh lưu trữ trùng lặp không cần thiết cho các dataset lớn |
| Thêm lifecycle cleanup rule | Giảm chi phí dài hạn từ các noncurrent version và delete markers |
| Không dùng custom domain và WAF trong lab | Hữu ích cho production, không cần thiết cho FCAJ service validation ngắn hạn |
| Giữ bí mật các ảnh chụp màn hình | Ngăn AWS account identifier và request identifier đưa vào báo cáo công khai |

## Quyết định Vị trí Dữ liệu

| Loại dữ liệu | Quyết định lưu trữ Tuần 6 | Lý do |
| --- | --- | --- |
| Log Zeek/Suricata thô | S3 private objects | File đối tượng lớn, không phải hàng truy vấn |
| CSV dataset | S3 private objects | Phù hợp cho batch processing và AI pipeline sau này |
| Model artifact | S3 private objects | Object storage phù hợp cho model file và versioned output |
| File frontend/report tĩnh | S3 website files và CloudFront | Nội dung tĩnh phục vụ trình duyệt |
| Bản ghi alert cuối cùng | RDS PostgreSQL | Cần filtering, sorting, query và dashboard persistence |

## Vấn đề và Khắc phục

| Vấn đề | Giải pháp |
| --- | --- |
| Bật S3 static website endpoint không tự động làm các đối tượng thành public | Tải lên `index.html` và `error.html`, sau đó cấu hình bucket policy cho các file đó |
| URL đối tượng trực tiếp của raw dataset trả về AccessDenied | Xử lý đây là hành vi mong đợi vì raw log nên được giữ riêng tư |
| Bucket policy rộng sẽ làm lộ các đối tượng raw dataset | Sử dụng danh sách tài nguyên chỉ cho `index.html` và `error.html` |
| S3 website endpoint dùng HTTP | Thêm CloudFront để có HTTPS viewer access qua CloudFront domain |
| Lựa chọn CloudFront origin có thể gây nhầm lẫn | Sử dụng S3 website endpoint vì lab này phụ thuộc vào hành vi static website hosting |
| Versioning có thể làm tăng chi phí lưu trữ | Thêm lifecycle rule cho các phiên bản cũ và delete markers trong demo prefix |
| Lifecycle cleanup không xảy ra ngay lập tức | Ghi lại rằng lifecycle processing là bất đồng bộ và dựa trên tuổi đời đối tượng |

## Bài học rút ra

S3 là object store, không phải hệ thống folder truyền thống. Các prefix như `raw/zeek/` là một phần của object key và giúp tổ chức project artifact một cách logic.

Quyền truy cập công khai nên được thiết kế theo từng trường hợp sử dụng. Telemetry thô và dataset nên giữ riêng tư, trong khi các file website được chọn lọc có thể được công khai cho lab website tĩnh.

S3 static website hosting và phân phối CloudFront với private origin là hai kiến trúc khác nhau. Lab Tuần 6 sử dụng mô hình website endpoint, vì vậy báo cáo không nên khẳng định private-bucket CloudFront access nếu chưa có bằng chứng riêng biệt.

Versioning hữu ích nhưng không miễn phí. Mỗi noncurrent object version có thể làm tăng chi phí lưu trữ, vì vậy lifecycle rule là kiểm soát bổ sung quan trọng.

## Kết quả đạt được trong tuần

Vào cuối Tuần 6, dự án đã có một mô hình lưu trữ S3 và phân phối CloudFront được xác minh.

Lab cho thấy cách tải lên dữ liệu SOC-style thô lên S3, giữ dữ liệu đó riêng tư, công khai các file website được chọn lọc, phục vụ trang tĩnh qua CloudFront, và sử dụng Versioning cùng Lifecycle rules để quản lý lịch sử đối tượng và dọn dẹp.

Điều này tạo ra nền tảng lưu trữ vững chắc hơn cho các SOC dataset, frontend artifact, báo cáo và model file trong tương lai.

## Kế hoạch tuần tiếp theo

Tuần 7 sẽ rời khỏi các AWS managed services và tập trung vào kiến trúc IDS lab cục bộ (local IDS lab architecture) tạo ra security telemetry thực tế cho dự án.

Trọng tâm sẽ là:

- Phân đoạn mạng pfSense
- Nguồn telemetry Suricata và Zeek
- Tạo attack traffic bằng Kali
- Workload Ubuntu nạn nhân
- ELK hoặc quy trình xem xét log
- Sơ đồ luồng traffic và bằng chứng lab
