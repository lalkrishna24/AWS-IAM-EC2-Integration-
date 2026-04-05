# 🔐 AWS IAM & EC2 Integration 

> **Real-world implementation of AWS Identity and Access Management (IAM) — covering IAM Users, Roles, Policies, and secure EC2-to-S3 access using role-based authentication without storing credentials.**

---

## 📌 Overview

This repository documents my hands-on practice with **AWS IAM** and **EC2-S3 integration**, where I explored the three core access scenarios in cloud environments:

| Scenario | Description |
|---|---|
| **User → Service** | IAM User directly accessing AWS services |
| **Service → Service** | EC2 accessing S3 using an attached IAM Role |
| **User → Service → Service** | User triggers EC2 which securely accesses S3 |

All tasks were performed on **Amazon Linux 2023** EC2 instances in `ap-south-1` (Mumbai).

---

## 🧰 Tech Stack & Tools

| Category | Details |
|---|---|
| Cloud Platform | AWS (IAM, EC2, S3) |
| OS | Amazon Linux 2023 |
| CLI | AWS CLI v2 |
| Access Method | EC2 Instance Connect (browser SSH) |
| Region | ap-south-1 (Mumbai) |

---

## ✅ What I Implemented

### 1. 👤 IAM Configuration

#### Created IAM Users
- Created IAM users with programmatic access (Access Key + Secret Key)
- Attached custom policies following the **least privilege model**
- Tested access with and without specific permissions

#### Created IAM Roles
- Created a role with `AmazonS3ReadOnlyAccess` policy
- Set the **trusted entity** as `EC2` (so EC2 instances can assume the role)
- Attached the role to specific EC2 instances

#### IAM Policy Used (S3 Read Access)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### 2. 🖥️ EC2 Instance Setup

- Launched **3 Amazon Linux 2023** EC2 instances for testing different scenarios:
  - `EC2 - without Role` → to demonstrate access denial
  - `EC2 - With Role` → to demonstrate successful role-based access
  - `EC2 - without role (configured via CLI)` → to demonstrate credential-based access

- Connected to all instances via **EC2 Instance Connect**

---

### 3. 🧩 Access Scenarios – Implemented & Tested

---

#### ❌ Scenario 1 — EC2 Without Role (Access Denied)

No IAM Role attached. Running `aws s3 ls` fails immediately:

```bash
[ec2-user@ip-172-31-42-191 ~]$ aws s3 ls
Unable to locate credentials. You can configure credentials by running "aws login".
```

> **Root Cause:** Without an IAM Role or configured credentials, the EC2 instance has no identity to authenticate AWS API calls.

**Screenshot — EC2 Without Role (Credential Error):**

![EC2 without role - credential error](https://raw.githubusercontent.com/lalkrishna24/AWS-IAM-EC2-Integration-/01dc0afede99e8ed2af8fd356af9cad0d19e9c62/1771071843952.jpeg)

---

#### ✅ Scenario 2 — EC2 With IAM Role Attached (Secure Access)

Attached an IAM Role (`S3ReadAccess`) directly to the EC2 instance. No credentials stored anywhere. `aws s3 ls` works immediately using **temporary credentials** from the EC2 metadata service:

```bash
[ec2-user@ip-172-31-32-131 ~]$ aws s3 ls
2026-02-13 14:04:19 lal-1
2026-02-14 10:45:36 lal-2
2026-02-14 10:45:55 lal-3
2026-02-14 10:46:11 lal-4
2026-02-14 10:46:27 lal-5
2026-02-14 10:46:44 lal-6
```

> ✅ **Best Practice:** Always use IAM Roles for EC2 instead of storing access keys. Roles use short-lived temporary credentials rotated automatically by AWS STS.

**Screenshot — EC2 With IAM Role (Successful S3 List):**

![EC2 with IAM role - S3 access](https://raw.githubusercontent.com/lalkrishna24/AWS-IAM-EC2-Integration-/01dc0afede99e8ed2af8fd356af9cad0d19e9c62/1771071843524.jpeg)

---

#### 🔑 Scenario 3 — EC2 Without Role, Configured via `aws configure`

To understand the alternative (and less secure) approach, I manually configured AWS credentials using `aws configure` on an instance without a role:

```bash
[ec2-user@ip-172-31-40-129 ~]$ aws s3 ls
Unable to locate credentials. You can configure credentials by running "aws login".

[ec2-user@ip-172-31-40-129 ~]$ aws configure
AWS Access Key ID [None]: ******************
AWS Secret Access Key [None]: ******************
Default region name [None]: ap-south-1
Default output format [None]: json

[ec2-user@ip-172-31-40-129 ~]$ aws s3 ls
2026-02-13 14:04:19 lal-1
2026-02-14 10:45:36 lal-2
2026-02-14 10:45:55 lal-3
2026-02-14 10:46:11 lal-4
2026-02-14 10:46:27 lal-5
2026-02-14 10:46:44 lal-6
```

> ⚠️ **Security Warning:** Hardcoding credentials via `aws configure` stores them in `~/.aws/credentials` as plain text. This approach is **not recommended** for production. Always prefer IAM Roles.

**Screenshot — EC2 Without Role, Configured via AWS CLI:**

![EC2 aws configure credentials](https://raw.githubusercontent.com/lalkrishna24/AWS-IAM-EC2-Integration-/01dc0afede99e8ed2af8fd356af9cad0d19e9c62/1771071845400.jpeg)

---

#### 🚫 Scenario 4 — S3 Console Access Without Permissions

Logged into the AWS Console with an IAM User that lacked `s3:ListAllMyBuckets`. The S3 console shows:

> *"You don't have permissions to list buckets"*

**Screenshot — S3 Console Permission Denied:**

![S3 console no permission](https://raw.githubusercontent.com/lalkrishna24/AWS-IAM-EC2-Integration-/01dc0afede99e8ed2af8fd356af9cad0d19e9c62/1771071845554.jpeg)

> **Fix:** Attach the correct IAM policy to the user/role granting `s3:ListAllMyBuckets`.

---

### 4. 🔐 Security Best Practices Applied

| Practice | Status |
|---|---|
| Used IAM Roles instead of Access Keys on EC2 | ✅ Done |
| Followed Least Privilege for IAM Policies | ✅ Done |
| Tested access denial before granting access | ✅ Done |
| Avoided storing long-term credentials on EC2 | ✅ Done |
| Configured Security Groups properly | ✅ Done |

---

## 📊 Instance Summary

| Instance | ID | Public IP | Private IP | Role Attached |
|---|---|---|---|---|
| EC2 - without Role | `i-0ec257a510e306752` | 43.205.254.11 | 172.31.42.191 | ❌ No |
| EC2 - With Role | `i-095b9b07dcf6471d5` | 13.127.112.200 | 172.31.32.131 | ✅ Yes |
| EC2 - without role (CLI config) | `i-007b673f7c5db4e2c` | 13.126.31.164 | 172.31.40.129 | ❌ No |

---

## 📁 Repository Structure

```
aws-iam-ec2-s3-integration/
│
├── README.md                              # This file
│
├── iam/
│   ├── s3-read-policy.json                # IAM Policy for S3 read access
│   ├── ec2-trust-policy.json              # Trust policy for EC2 role
│   └── iam-setup-notes.md                 # IAM creation steps
│
├── ec2/
│   ├── launch-instance.md                 # EC2 launch steps
│   ├── attach-role.md                     # How to attach IAM Role to EC2
│   └── aws-cli-commands.sh                # CLI commands used
│
└── screenshots/
    ├── 1771071843952.jpeg        # EC2 without role - credential error
    ├── 1771071843524.jpeg        # EC2 with IAM role - S3 access
    ├── 1771071845400.jpeg        # EC2 aws configure S3 access
    └── 1771071845554.jpeg        # S3 console permission denied
```

---

## 🔧 Key AWS CLI Commands Used

```bash
# Check current identity
aws sts get-caller-identity

# List S3 buckets
aws s3 ls

# Configure credentials manually (not recommended for production)
aws configure

# List IAM roles
aws iam list-roles

# Get metadata token (IMDSv2 — how roles work under the hood)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Retrieve temporary credentials from instance metadata
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Copy a file to S3
aws s3 cp myfile.txt s3://my-bucket-name/

# Sync a folder to S3
aws s3 sync ./myfolder s3://my-bucket-name/myfolder/
```

---

## 💡 Key Learnings

- **IAM Roles > Access Keys** on EC2: Roles use STS temporary credentials (auto-rotated every hour) — no credentials stored on disk
- **Credential chain order**: AWS CLI checks in this order: environment variables → `~/.aws/credentials` → EC2 instance metadata (role)
- **Least privilege matters**: Even `s3:ListAllMyBuckets` must be explicitly granted — it is NOT included by default
- **Trust policy is the glue**: An IAM Role only works on EC2 if the trust policy explicitly allows `ec2.amazonaws.com` to assume the role
- **`aws configure` vs IAM Role**: Both work, but `aws configure` stores long-lived static credentials — a security risk if the instance is compromised
- **S3 is global but region-scoped**: `aws s3 ls` works regardless of region but the CLI region config affects latency and endpoint routing

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                     AWS Account                         │
│                                                         │
│  ┌──────────┐    assumes    ┌─────────────────────┐    │
│  │  EC2     │─────role─────▶│  IAM Role           │    │
│  │ Instance │               │  (S3ReadAccess)     │    │
│  └──────────┘               └────────┬────────────┘    │
│       │                              │ grants           │
│       │                              ▼                  │
│       │                    ┌─────────────────────┐     │
│       └────────────────────▶  Amazon S3 Buckets  │     │
│            aws s3 ls        └─────────────────────┘     │
│                                                         │
│  ✅ No credentials stored on EC2                        │
│  ✅ Temporary credentials via instance metadata         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔗 References

- [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
- [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [AWS CLI Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [S3 Access Control](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-overview.html)
- [EC2 Instance Metadata & Credentials](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#instance-metadata-security-credentials)

---

## 🏷️ Tags

`AWS` `IAM` `EC2` `S3` `IAM Roles` `IAM Policies` `Cloud Security` `DevOps` `Amazon Linux` `AWS CLI` `Role-Based Access Control` `RBAC` `Least Privilege` `ap-south-1` `Hands-On`

---

> 🌐 *All experiments were performed on live AWS EC2 instances in the Asia Pacific (Mumbai) region. Credentials visible in screenshots have been rotated and are no longer active.*
