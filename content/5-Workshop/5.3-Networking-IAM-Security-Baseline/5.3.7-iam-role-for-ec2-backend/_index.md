---
title: "IAM Role for EC2 Backend"
date: 2026-07-08
weight: 7
chapter: false
pre: " <b> 5.3.7. </b> "
---

## Introduction

IAM Role is used to grant permissions for the **EC2 Backend** to access AWS services such as S3, SQS, Secrets Manager, and CloudWatch.

In the **Hybrid SOC on AWS** system, the Backend EC2 should not use AWS access keys or secret keys hard-coded in source code or `.env` files. Instead, the backend should use an IAM Role directly attached to the EC2 instance.

## Step 1: Go to IAM Roles Page

On the AWS Console, open the **IAM** service, then select **Roles**.

Find the backend role of the project:
```text
socai-backend-role
```

If the actual role name on AWS is different, you need to write the exact real name from the console.

![IAM Backend Role Summary](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-07-01-iam-backend-role-summary.jpg)

## Step 2: Check Role Summary

On the role summary page, you need to check the main information:

```text
Role name: socai-backend-role
Trusted entity: EC2
Use case: EC2 instance
```

This role will be attached to the Backend EC2 so that the backend can call AWS services without needing an access key.

The permissions required by the backend include:
```text
S3
SQS
Secrets Manager
CloudWatch Logs
KMS if secret or bucket uses a custom KMS key
```

## Step 3: Check Permissions

Open the **Permissions** tab to check the policy attached to the role.

The backend IAM Role should have minimum permissions to:
- Send messages to SQS.
- Write raw evidence to the S3 Data Bucket.
- Read the RDS connection secret if the backend uses Secrets Manager.
- Write logs to CloudWatch if logging is configured.
- Decrypt KMS if secret or S3 object uses a KMS custom key.

Permissions to check:
```text
sqs:SendMessage
s3:PutObject
s3:GetObject
secretsmanager:GetSecretValue
logs:CreateLogStream
logs:PutLogEvents
kms:Decrypt
```

![IAM Backend Permissions](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-07-02-iam-backend-permissions.jpg)

## Step 4: Check Trust Relationship

Open the **Trust relationships** tab to check which service is allowed to assume this role.

The trust policy should have a principal of EC2:
```text
ec2.amazonaws.com
```

If the trust relationship is incorrect, the EC2 instance will not be able to use this role.

Example trust policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

![IAM Trust Policy EC2](/images/5-Workshop/5.3-Networking-IAM-Security-Baseline/w-trang-07-03-iam-trust-policy-ec2.jpg)

## Step 5: Check if the Role is Attached to EC2 Backend

After checking the IAM Role, you need to go back to the EC2 instance `socai-dev-backend-01` to confirm that the role has been attached to the instance.

How to check:
```text
EC2
-> Instances
-> Select socai-dev-backend-01
-> Security tab
-> IAM Role
```

The IAM Role must be the project's backend role, for example:
```text
socai-backend-role
```

If EC2 is not attached to an IAM Role, the backend may encounter errors:
```text
Unable to locate credentials
AccessDenied
```
when calling S3, SQS, or Secrets Manager.

## IAM Role in the Workshop

| Component | Role |
|---|---|
| EC2 Backend | Runs FastAPI service |
| IAM Role | Grants permissions for EC2 to call AWS services |
| S3 | Stores raw evidence |
| SQS | Receives event messages from backend |
| Secrets Manager | Stores database connection secrets |
| CloudWatch | Writes logs and monitors runtime |
| KMS | Decrypts secrets or objects if configured |

IAM Role helps the backend call AWS services more safely, without needing to store credentials directly on the server.

## Notes

Do not include the following information in screenshots or markdown:
- AWS access key.
- AWS secret key.
- Secret value.
- DB password.
- Private key `.pem`.
- Token or API key.
- Un `.env` content.

If the policy has a resource ARN containing an account ID, you can crop or redact the account ID if not necessary.

In a production environment, IAM Role should be granted permissions following the **least privilege** principle, only granting the exact actions and exact resources that the backend needs.

## Conclusion

After this step, the implementer should ensure:

- EC2 Backend uses an IAM Role instead of an access key.
- The backend role has a trusted entity of EC2.
- The role has the necessary permissions to call S3, SQS, Secrets Manager, and CloudWatch.
- If using a KMS custom key, the role must have the `kms:Decrypt` permission.
- The IAM Role has been attached to EC2 Backend.
- No sensitive credentials are hard-coded in source code or `.env`.

After completing this section, the network and security baseline for Backend EC2 is ready to continue checking the backend runtime and event processing pipeline.

The next section will check **Backend EC2 Runtime**.
