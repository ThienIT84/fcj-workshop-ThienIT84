---
title: "Backend EC2 Runtime"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Introduction

This section checks the **Backend EC2 Runtime of the Hybrid SOC system on AWS. The EC2 Backend is where the FastAPI service runs, receives requests from the dashboard or collector, then prepares data to send to the SQS, S3, and RDS pipeline.

In this section, we will check:

- EC2 Backend instance.
- Running status of the instance.
- Connecting to EC2.
- Python runtime and dependencies.
- systemd service.
- Health endpoint `/health`.
-  for async endpoint after backend source is complete.

## Step 1: Check EC2 Instances List

On the AWS Console, open the **EC2** service, then select **Instances**.

In the current project, the backend instance is:

```text
socai-dev-backend-01
```

In addition to the backend instance, the project also has a worker instance:

```text
socai-dev-ai-worker-01
```

The image below shows the list of EC2 instances in the project.

![EC2 Instance List](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-01-ec2-instance-list.jpg)

## Step 2: Check Backend EC2 Status

Select instance `socai-dev-backend-01` to check detailed information.

Current backend instance info:

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

Instance does not have Public IPv4, which means the backend is not directly exposed to the Internet. This is appropriate when the backend is in a private network and receives traffic through ALB or appropriate network layers.

![EC2 Backend Summary](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-02-ec2-backend-summary.jpg)

## Step 3: Connect to EC2 Backend

After confirming the instance is running, select **Connect** to access EC2.

You can use one of the following methods:

```text
EC2 Instance Connect
SSH client
Session Manager if SSM is configured
```

In this workshop, the terminal has successfully accessed EC2 and shows the prompt:

```text
sh-5.2$
```

This proves that the implementer has connected to the backend server.

![EC2 Connect Success](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-03-ec2-connect-success.jpg)

## Step 4: Check Basic Commands on EC2

After accessing EC2, run some basic commands to check the environment:

```bash
pwd
ls -la
```

The current result shows that commands have run successfully on EC2.

![Basic Command Check](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-04-basic-command-check.jpg)

Note: this image proves access and operations have been able to operate on EC2. If you need to go to the correct backend source directory and run additional commands to prove more detailed backend source:

```bash
pwd
ls -la
git status
```

## Step 5: Check FastAPI and Uvicorn Runtime

Check backend runtime with command:

```bash
python3 -m pip freeze | grep -E "fastapi|uvicorn|boto3|botocore"
```

At first check, backend already has main packages to run the FastAPI service:

```text
fastapi==0.128.0
uvicorn==0.39.0
```

Where:

- `fastapi`: framework used to build backend API.
- `uvicorn`: ASGI server used to run FastAPI.

![Runtime FastAPI Uvicorn](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-05-runtime-fastapi-uvicorn.jpg)

## Step 6: Check boto3 and botocore Dependencies

After installing or updating dependencies, check again with command:

```bash
python3 -m pip freeze | grep -E "fastapi|uvicorn|boto3|botocore"
```

Current results show that the backend runtime has all the important packages:

```text
boto3==1.42.97
botocore==1.42.97
fastapi==0.128.0
uvicorn==0.39.0
```

Where:

- `boto3`: AWS SDK for Python, used for backend to call AWS services.
- `botocore`: foundational dependency of boto3.
- `fastapi`: backend API framework.
- `uvicorn`: server running FastAPI.

`boto3` and `botocore` are needed for the backend to integrate with S3 and SQS.

![Backend Dependencies Boto3](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-06-backend-dependencies-boto3.jpg)

## Step 7: Check systemd service

Backend is running with `systemd` with service name:

```text
socai-backend
```

Run the following command to check service status:

```bash
sudo systemctl status socai-backend --no-pager
```

Current results show:

```text
socai-backend.service
Loaded: loaded
enabled
Active: active (running)
```

This proves that the backend service is running stably on EC2. The systemd logs also show that the backend receives `/health` request and returns `200 OK`.

![Systemctl Backend Active](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-07-systemctl-backend-active.jpg)

## Step 8: Test health endpoint

The `/health` endpoint checks if the backend process is alive and FastAPI is accepting requests on port `8000`.

Run the following command directly on EC2:

```bash
curl -i http://127.0.0.1:8000/health
```

The returned result:

```text
HTTP/1.1 200 OK
server: uvicorn
content-type: application/json
```

This proves that the `socai-backend` service is running successfully on EC2.

![Health Local Response](/images/5-Workshop/5.5-Backend-EC2-Runtime/w-trang-04-08-health-local-response.jpg)

Since the backend instance currently does not have Public IPv4, so health check is tested locally within EC2. If later the backend is exposed via ALB or CloudFront, you can test:

```bash
curl -i http://<ALB_DNS_NAME>/health
curl -i https://<CLOUDFRONT_DOMAIN>/health
```

## Step 9:  for async ingestion endpoint

The `/api/events/http/async endpoint receives HTTP evidence from the collector or tailer.

When the backend source is complete, this endpoint will perform the following steps:

1. Validate payload.
2. Write raw evidence to S3 Data Bucket.
3. Send message to SQS.
4. Return `event_id` and `s3_raw_key`.

Currently, this part will be updated after the backend source has complete boto3 integration for S3 and SQS.

Sample expected curl:

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

Expected after source runs correctly:

```json
{
  "status": "queued",
  "event_id": "evt-xxxx",
  "s3_raw_key": "raw/zeek/http/2026/07/06/evt-xxxx.json",
  "queue": "socai-events-queue"
}
```

## Troubleshooting

| Error | Symptoms | Common Cause | Solution |
|---|---|---|---|
| Cannot connect to EC2 | Timeout or connection error | Wrong connect method, subnet or Security Group | Check EC2 Connect, Security Group and subnet |
| `/health` doesn't return 200 | Connection refused | Backend not running or wrong port | Check `systemctl status`, `journalctl`, backend port |
| Missing dependencies | Import error or service not starting | Required packages not installed | Reinstall requirements or check pip environment |
| `AccessDenied` when calling SQS/S3 | API returns 500 or AccessDenied logs | IAM Role missing permissions | Check IAM Role and policy |
| `Unable to locate credentials` | boto3 cannot find credentials | IAM Role not attached to EC2 | Attach IAM Role to EC2 |
| Public test cannot access | Local curl timeout | Backend does not have public IP or no ALB | Test locally within EC2 or configure ALB |
| DB connection fail | Connection timeout logs | Wrong RDS SG, subnet or secret | Check RDS SG, secret and backend env |

## Conclusion

After this section, the implementer has confirmed:

- EC2 Backend `socai-dev-backend-01` is running.
- Instance ID is `i-06f8f2f017439493d`.
- Instance type is `t3.large`.
- Instance is in AZ `ap-southeast-1a`.
- Private IPv4 is `10.20.143.156`.
- Instance has no Public IPv4.
- Able to connect to EC2 backend server.
- Backend runtime has FastAPI, Uvicorn, boto3, and botocore.
- Service `socai-backend` is `active (running)`.
- Endpoint `/health` returns `HTTP/1.1 200 OK`.
- Async endpoint will be updated after S3/SQS source is complete.
