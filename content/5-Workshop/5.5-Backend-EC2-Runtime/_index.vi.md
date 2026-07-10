---
title: "Backend EC2 Runtime"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Giới thiệu

Phần này kiểm tra **Backend EC2 Runtime của hệ thống Hybrid SOC trên AWS**. EC2 Backend là nơi chạy dịch vụ FastAPI, nhận request từ dashboard hoặc collector, sau đó chuẩn bị dữ liệu để gửi vào pipeline SQS, S3 và RDS.

Trong phần này, chúng ta sẽ kiểm tra:
- EC2 Backend instance.
- Trạng thái chạy của instance.
- Kết nối đến EC2.
- Python runtime và dependencies.
- systemd service.
- Endpoint health `/health`.
-  cho async endpoint sau khi hoàn thành backend source.

## Bước 1: Kiểm tra danh sách EC2 Instances

Trên AWS Console, mở dịch vụ **EC2**, sau đó chọn **Instances**.

Trong dự án hiện tại, backend instance là:
```text
socai-dev-backend-01
```

Ngoài backend instance, dự án cũng có worker instance:
```text
socai-dev-ai-worker-01
```

Hình dưới đây cho thấy danh sách EC2 instances trong dự án.

![EC2 Instance List](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-01-ec2-instance-list.jpg)

## Bước 2: Kiểm tra trạng thái Backend EC2

Chọn instance `socai-dev-backend-01` để kiểm tra thông tin chi tiết.

Thông tin backend instance hiện tại:
```text
Instance name: socai-dev-backend-01
Instance ID: i-06f8f2f017439493d
Instance state: Running
Instance type: t3.large
Status check: 3/3 checks passed
Availability Zone: ap-southeast-1a
Private IPv4 address: 10.20.143.156
Public IPv4 address: -
```

Instance không có Public IPv4, điều này có nghĩa là backend không được expose trực tiếp ra Internet. Điều này phù hợp khi backend nằm trong mạng riêng và nhận traffic thông qua ALB hoặc các lớp mạng phù hợp.

![EC2 Backend Summary](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-02-ec2-backend-summary.jpg)

## Bước 3: Kết nối đến EC2 Backend

Sau khi xác nhận instance đang chạy, chọn **Connect** để truy cập EC2.

Bạn có thể sử dụng một trong các phương pháp sau:
```text
EC2 Instance Connect
SSH client
Session Manager nếu đã cấu hình SSM
```

Trong workshop này, terminal đã truy cập EC2 thành công và hiển thị prompt:
```text
sh-5.2$
```

Điều này chứng minh người thực hiện đã kết nối được đến backend server.

![EC2 Connect Success](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-03-ec2-connect-success.jpg)

## Bước 4: Kiểm tra lệnh cơ bản trên EC2

Sau khi truy cập EC2, chạy một số lệnh cơ bản để kiểm tra môi trường:

```bash
pwd
ls -la
```

Kết quả hiện tại cho thấy các lệnh đã chạy thành công trên EC2.

![Basic Command Check](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-04-basic-command-check.jpg)

Lưu ý: hình này chứng minh truy cập và thao tác đã có thể hoạt động trên EC2. Nếu cần đến đúng thư mục backend source và chạy thêm lệnh để chứng minh backend source chi tiết hơn:
```bash
pwd
ls -la
git status
```

## Bước 5: Kiểm tra FastAPI và Uvicorn Runtime

Kiểm tra backend runtime bằng lệnh:

```bash
python3 -m pip freeze | grep -E "fastapi|uvicorn|boto3|botocore"
```

Ở lần kiểm tra đầu tiên, backend đã có các package chính để chạy dịch vụ FastAPI:
```text
fastapi==0.128.0
uvicorn==0.39.0
```

Trong đó:
- `fastapi`: framework dùng để xây dựng backend API.
- `uvicorn`: ASGI server dùng để chạy FastAPI.

![Runtime FastAPI Uvicorn](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-05-runtime-fastapi-uvicorn.jpg)

## Bước 6: Kiểm tra boto3 và botocore Dependencies

Sau khi cài đặt hoặc cập nhật dependencies, kiểm tra lại bằng lệnh:

```bash
python3 -m pip freeze | grep -E "fastapi|uvicorn|boto3|botocore"
```

Kết quả hiện tại cho thấy backend runtime đã có đầy đủ các package quan trọng:
```text
boto3==1.42.97
botocore==1.42.97
fastapi==0.128.0
uvicorn==0.39.0
```

Trong đó:
- `boto3`: AWS SDK for Python, dùng để backend gọi các dịch vụ AWS.
- `botocore`: dependency nền của boto3.
- `fastapi`: backend API framework.
- `uvicorn`: server chạy FastAPI.

`boto3` và `botocore` cần thiết để backend tích hợp với S3 và SQS.

![Backend Dependencies Boto3](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-06-backend-dependencies-boto3.jpg)

## Bước 7: Kiểm tra systemd service

Backend đang chạy với `systemd` với tên service:
```text
socai-backend
```

Chạy lệnh sau để kiểm tra trạng thái service:
```bash
sudo systemctl status socai-backend --no-pager
```

Kết quả hiện tại cho thấy:
```text
socai-backend.service
Loaded: loaded
enabled
Active: active (running)
```

Điều này chứng minh backend service đang chạy ổn định trên EC2. Các log systemd cũng cho thấy backend nhận request `/health` và trả về `200 OK`.

![Systemctl Backend Active](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-07-systemctl-backend-active.jpg)

## Bước 8: Kiểm tra health endpoint

Endpoint `/health` kiểm tra xem tiến trình backend còn sống hay không và FastAPI có đang chấp nhận request trên cổng `8000` hay không.

Chạy lệnh sau trực tiếp trên EC2:
```bash
curl -i http://127.0.0.1:8000/health
```

Kết quả trả về:
```text
HTTP/1.1 200 OK
server: uvicorn
content-type: application/json
```

Điều này chứng minh service `socai-backend` đang chạy thành công trên EC2.

![Health Local Response](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-08-health-local-response.jpg)

Vì backend instance hiện tại không có Public IPv4, nên health check được test cục bộ trong EC2. Nếu sau này backend được expose qua ALB hoặc CloudFront, bạn có thể test:
```bash
curl -i http://<ALB_DNS_NAME>/health
curl -i https://<CLOUDFRONT_DOMAIN>/health
```

## Bước 9:  cho async ingestion endpoint

Endpoint `/api/events/http/async` nhận HTTP evidence từ collector hoặc tailer.

Khi backend source hoàn thành, endpoint này sẽ thực hiện các bước sau:
1. Validate payload.
2. Ghi raw evidence vào S3 Data Bucket.
3. Gửi message đến SQS.
4. Trả về `event_id` và `s3_raw_key`.

Hiện tại, phần này sẽ được cập nhật sau khi backend source hoàn thành tích hợp boto3 cho S3 và SQS.

Ví dụ curl mong đợi:
```bash
curl -i -X POST "http://<BACKEND_ENDPOINT>:8000/api/events/http/async" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "zeek",
    "log_type": "http",
    "timestamp": "2026-07-06T10:05:00Z",
    "src_ip": "192.168.56.10",
    "dst_ip": "192.168.56.20",
    "method": "GET",
    "uri": "/login.php?id=1 OR 1=1",
    "user_agent": "sqlmap"
  }'
```

Mong đợi sau khi source chạy đúng:
```json
{
  "status": "queued",
  "event_id": "evt-xxxx",
  "s3_raw_key": "raw/zeek/http/2026/07/06/evt-xxxx.json",
  "queue": "socai-events-queue"
}
```

## Troubleshooting

| Lỗi | Triệu chứng | Nguyên nhân thường gặp | Giải pháp |
|---|---|---|---|
| Không kết nối được EC2 | Timeout hoặc lỗi kết nối | Sai phương pháp kết nối, subnet hoặc Security Group | Kiểm tra EC2 Connect, Security Group và subnet |
| `/health` không trả về 200 | Connection refused | Backend không chạy hoặc sai cổng | Kiểm tra `systemctl status`, `journalctl`, backend port |
| Thiếu dependencies | Lỗi import hoặc service không khởi động | Chưa cài đặt các package cần thiết | Cài lại requirements hoặc kiểm tra môi trường pip |
| `AccessDenied` khi gọi SQS/S3 | API trả về 500 hoặc log AccessDenied | IAM Role thiếu permissions | Kiểm tra IAM Role và policy |
| `Unable to locate credentials` | boto3 không tìm được credentials | IAM Role chưa gắn vào EC2 | Gắn IAM Role vào EC2 |
| Không thể truy cập từ bên ngoài | Local curl timeout | Backend không có public IP hoặc chưa có ALB | Test cục bộ trong EC2 hoặc cấu hình ALB |
| Kết nối DB thất bại | Log timeout kết nối | Sai RDS SG, subnet hoặc secret | Kiểm tra RDS SG, secret và backend env |

## Kết luận

Sau phần này, người thực hiện đã xác nhận:
- EC2 Backend `socai-dev-backend-01` đang chạy.
- Instance ID là `i-06f8f2f017439493d`.
- Instance type là `t3.large`.
- Instance nằm ở AZ `ap-southeast-1a`.
- Private IPv4 là `10.20.143.156`.
- Instance không có Public IPv4.
- Có thể kết nối đến backend server EC2.
- Backend runtime có FastAPI, Uvicorn, boto3 và botocore.
- Service `socai-backend` ở trạng thái `active (running)`.
- Endpoint `/health` trả về `HTTP/1.1 200 OK`.
- Async endpoint sẽ được cập nhật sau khi hoàn thành source S3/SQS.
