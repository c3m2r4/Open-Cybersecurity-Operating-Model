# AWS Cloud Security Lab — LocalStack Simulation Playbook

> **Classification:** Internal Lab Documentation  
> **Environment:** LocalStack v3.x (AWS Emulation) | Kali Linux  
> **Status:** ✅ Completed  
> **Last Updated:** 2026-09-19  
> **Author:** camara  

---

## Table of Contents

1. [[#Overview]]
2. [[#Prerequisites]]
3. [[#Environment Setup]]
4. [[#Phase 1 — S3 Bucket Operations]]
5. [[#Phase 2 — IAM Identity & Access Management]]
6. [[#Phase 3 — CloudTrail Audit Logging]]
7. [[#Phase 4 — Detection & Verification]]
8. [[#Phase 5 — Public S3 Bucket Policy & Anonymous Access Testing]]
9. [[#Lab Findings & Observations]]
10. [[#Security Recommendations]]
11. [[#Key Takeaways]]

---

## Overview

This playbook documents a hands-on cloud security lab conducted against a **LocalStack** instance — a fully functional local AWS cloud stack used for offline testing, security research, and skills development.

The lab simulates real-world AWS workflows including:

- **S3 bucket provisioning** and object lifecycle management
- **IAM user creation**, policy definition, and privilege assignment
- **CloudTrail audit trail** configuration and logging verification
- **Detection engineering** — verifying what gets logged vs. what doesn't

> [!NOTE]
> All operations target `http://127.0.0.1:4566` (LocalStack endpoint). No real AWS infrastructure was used. Credentials shown are synthetic LocalStack test values.

---

## Prerequisites

| Requirement | Details |
|---|---|
| **AWS CLI** | v2.36.17+ |
| **Python** | 3.14.6+ |
| **LocalStack** | Running on `localhost:4566` |
| **Platform** | Kali Linux (amd64) |
| **Permissions** | Root or sudo access |

### Verify AWS CLI Installation

```bash
aws --version
# Expected: aws-cli/2.36.17 Python/3.14.6 Linux/...
```

---

## Environment Setup

### Configure Admin Credentials (LocalStack Root)

```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
```

> [!TIP]
> LocalStack accepts any credential values. The `test`/`test` pair is the standard convention for unauthenticated local development.

### Confirm LocalStack Connectivity

```bash
aws --endpoint-url=http://127.0.0.1:4566 s3 ls
# Empty output = LocalStack is running and S3 is reachable
```

---

## Phase 1 — S3 Bucket Operations

### 1.1 Create the Primary Data Bucket

```bash
aws --endpoint-url=http://127.0.0.1:4566 s3 mb s3://cyber-lab-data
```

**Output:**
```
make_bucket: cyber-lab-data
```

### 1.2 Verify Bucket Creation

```bash
aws --endpoint-url=http://127.0.0.1:4566 s3 ls
```

**Output:**
```
2026-09-19 00:03:07 cyber-lab-data
```

### 1.3 Upload a Sensitive Object

```bash
echo "CONFIDENTIAL CYBER LAB DATA" > ~/cloud-secret.txt

aws --endpoint-url=http://127.0.0.1:4566 \
    s3 cp ~/cloud-secret.txt s3://cyber-lab-data/
```

**Output:**
```
upload: ./cloud-secret.txt to s3://cyber-lab-data/cloud-secret.txt
```

### 1.4 List Bucket Contents

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 ls s3://cyber-lab-data/
```

**Output:**
```
2026-09-19 00:04:22    28    cloud-secret.txt
```

### 1.5 Download and Verify Object Integrity

```bash
# Verify file does not exist locally before download
cat /tmp/recovered-secret.txt
# cat: /tmp/recovered-secret.txt: No such file or directory

aws --endpoint-url=http://127.0.0.1:4566 \
    s3 cp s3://cyber-lab-data/cloud-secret.txt /tmp/recovered-secret.txt

cat /tmp/recovered-secret.txt
# CONFIDENTIAL CYBER LAB DATA
```

### 1.6 Inspect Bucket Metadata

```bash
# Get bucket region constraint
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-bucket-location \
    --bucket cyber-lab-data
```

**Output:**
```json
{
    "LocationConstraint": null
}
```

```bash
# Get bucket ACL
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-bucket-acl \
    --bucket cyber-lab-data
```

**Output:**
```json
{
    "Owner": {
        "DisplayName": "floci",
        "ID": "000000000000"
    },
    "Grants": [
        {
            "Grantee": {
                "DisplayName": "floci",
                "ID": "000000000000",
                "Type": "CanonicalUser"
            },
            "Permission": "FULL_CONTROL"
        }
    ]
}
```

```bash
# Attempt to retrieve bucket policy
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-bucket-policy \
    --bucket cyber-lab-data
```

**Output:**
```
aws: [ERROR]: NoSuchBucketPolicy — The bucket policy does not exist
```

> [!IMPORTANT]
> No bucket policy is applied. In a real environment, S3 buckets storing sensitive data **must** have explicit resource-based policies enforcing least privilege and encryption-at-rest requirements.

---

## Phase 2 — IAM Identity & Access Management

### 2.1 Audit Existing IAM State

```bash
# List existing IAM users
aws --endpoint-url=http://127.0.0.1:4566 iam list-users
# { "Users": [] }

# List existing IAM roles
aws --endpoint-url=http://127.0.0.1:4566 iam list-roles
# { "Roles": [] }

# List locally-scoped custom policies
aws --endpoint-url=http://127.0.0.1:4566 iam list-policies --scope Local
# { "Policies": [] }
```

### 2.2 Create a New IAM User

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam create-user \
    --user-name cloud-lab-user
```

**Output:**
```json
{
    "User": {
        "Path": "/",
        "UserName": "cloud-lab-user",
        "UserId": "AIDA397WZ85ZZ86KAG8K",
        "Arn": "arn:aws:iam::000000000000:user/cloud-lab-user",
        "CreateDate": "2026-09-19T04:07:49.474741+00:00"
    }
}
```

### 2.3 Define a Custom IAM Policy

Create a JSON policy document granting full S3 access:

```bash
cat > ~/s3-lab-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "LabS3FullAccess",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
EOF
```

> [!WARNING]
> `"Action": "s3:*"` with `"Resource": "*"` grants **unrestricted S3 access** across all buckets. This violates the principle of least privilege. In production, always scope actions and resources to specific ARNs.

### 2.4 Register the Policy in IAM

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam create-policy \
    --policy-name LabS3FullAccess \
    --policy-document file://$HOME/s3-lab-policy.json
```

**Output:**
```json
{
    "Policy": {
        "PolicyName": "LabS3FullAccess",
        "PolicyId": "ANPA7BK4F97M45RM0CF0",
        "Arn": "arn:aws:iam::000000000000:policy/LabS3FullAccess",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "IsAttachable": true,
        "CreateDate": "2026-09-19T04:08:23.051160+00:00",
        "UpdateDate": "2026-09-19T04:08:23.051160+00:00"
    }
}
```

### 2.5 Retrieve Policy ARN via JMESPath Query

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam list-policies \
    --scope Local \
    --query 'Policies[?PolicyName==`LabS3FullAccess`].Arn' \
    --output text
```

**Output:**
```
arn:aws:iam::000000000000:policy/LabS3FullAccess
```

### 2.6 Attach Policy to the IAM User

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam attach-user-policy \
    --user-name cloud-lab-user \
    --policy-arn arn:aws:iam::000000000000:policy/LabS3FullAccess
```

### 2.7 Verify Policy Attachment

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam list-attached-user-policies \
    --user-name cloud-lab-user
```

**Output:**
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "LabS3FullAccess",
            "PolicyArn": "arn:aws:iam::000000000000:policy/LabS3FullAccess"
        }
    ]
}
```

### 2.8 Inspect Policy Document Version

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam get-policy-version \
    --policy-arn arn:aws:iam::000000000000:policy/LabS3FullAccess \
    --version-id v1
```

**Output:**
```json
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "LabS3FullAccess",
                    "Effect": "Allow",
                    "Action": "s3:*",
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2026-09-19T04:08:23.051160+00:00"
    }
}
```

### 2.9 Generate Access Keys for the IAM User

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    iam create-access-key \
    --user-name cloud-lab-user
```

**Output:**
```json
{
    "AccessKey": {
        "UserName": "cloud-lab-user",
        "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "Status": "Active",
        "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
        "CreateDate": "2026-09-19T04:10:42.574290+00:00"
    }
}
```

> [!CAUTION]
> **Never log or commit real AWS access keys.** These are synthetic LocalStack credentials and do not represent real AWS credentials. Always store secrets in AWS Secrets Manager or an approved vault.

### 2.10 Switch Identity — Assume IAM User Context

```bash
export AWS_ACCESS_KEY_ID='AKIAIOSFODNN7EXAMPLE'
export AWS_SECRET_ACCESS_KEY='wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'
export AWS_DEFAULT_REGION='us-east-1'
```

### 2.11 Verify IAM User Identity (STS)

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    sts get-caller-identity
```

**Output:**
```json
{
    "UserId": "000000000000",
    "Account": "000000000000",
    "Arn": "arn:aws:iam::000000000000:user/cloud-lab-user"
}
```

### 2.12 Validate S3 Access as cloud-lab-user

```bash
# List all buckets
aws --endpoint-url=http://127.0.0.1:4566 s3 ls

# List objects in target bucket
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 ls s3://cyber-lab-data/

# Download the confidential object
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 cp s3://cyber-lab-data/cloud-secret.txt \
    /tmp/cloud-lab-user-secret.txt

cat /tmp/cloud-lab-user-secret.txt
# CONFIDENTIAL CYBER LAB DATA
```

**Result:** `cloud-lab-user` successfully accessed and downloaded the object — permissions confirmed working.

---

## Phase 3 — CloudTrail Audit Logging

### 3.1 Audit Logging Baseline (Pre-Trail)

```bash
# No trails exist initially
aws --endpoint-url=http://127.0.0.1:4566 cloudtrail describe-trails
# { "trailList": [] }

aws --endpoint-url=http://127.0.0.1:4566 cloudtrail list-trails
# { "Trails": [] }
```

### 3.2 Restore Admin Context

```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
```

### 3.3 Create a Dedicated Log Bucket

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 mb s3://cloudtrail-lab-logs
```

**Output:**
```
make_bucket: cloudtrail-lab-logs
```

### 3.4 Create the CloudTrail Trail

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail create-trail \
    --name cloud-security-lab-trail \
    --s3-bucket-name cloudtrail-lab-logs
```

**Output:**
```json
{
    "Name": "cloud-security-lab-trail",
    "S3BucketName": "cloudtrail-lab-logs",
    "IncludeGlobalServiceEvents": true,
    "IsMultiRegionTrail": false,
    "TrailARN": "arn:aws:cloudtrail:us-east-1:000000000000:trail/cloud-security-lab-trail",
    "LogFileValidationEnabled": false,
    "IsOrganizationTrail": false
}
```

> [!IMPORTANT]
> `LogFileValidationEnabled: false` means log files are **not integrity-verified**. In production, always enable log file validation to detect tampering via SHA-256 digest chains.

### 3.5 Verify Trail Configuration

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail describe-trails
```

**Output:**
```json
{
    "trailList": [
        {
            "Name": "cloud-security-lab-trail",
            "S3BucketName": "cloudtrail-lab-logs",
            "IncludeGlobalServiceEvents": true,
            "IsMultiRegionTrail": false,
            "HomeRegion": "us-east-1",
            "TrailARN": "arn:aws:cloudtrail:us-east-1:000000000000:trail/cloud-security-lab-trail",
            "LogFileValidationEnabled": false,
            "HasCustomEventSelectors": false,
            "HasInsightSelectors": false,
            "IsOrganizationTrail": false
        }
    ]
}
```

### 3.6 Check Initial Logging Status

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail get-trail-status \
    --name cloud-security-lab-trail
```

**Output:**
```json
{
    "IsLogging": false
}
```

> [!WARNING]
> Trail is **created but not logging**. A CloudTrail trail must be explicitly started with `start-logging`. This is a common misconfiguration in real environments.

### 3.7 Enable Logging on the Trail

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail start-logging \
    --name cloud-security-lab-trail
```

### 3.8 Confirm Logging is Active

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail get-trail-status \
    --name cloud-security-lab-trail
```

**Output:**
```json
{
    "IsLogging": true,
    "StartLoggingTime": "2026-09-19T00:16:29.076000-04:00"
}
```

**Result:** CloudTrail is now actively logging.

---

## Phase 4 — Detection & Verification

### 4.1 Generate Auditable S3 Events (Post-Logging)

```bash
# Upload a test file (generates PutObject event)
echo "CLOUDTRAIL DETECTION TEST $(date)" > /tmp/cloudtrail-event-test.txt

aws --endpoint-url=http://127.0.0.1:4566 \
    s3 cp /tmp/cloudtrail-event-test.txt \
    s3://cyber-lab-data/
# upload: ../../tmp/cloudtrail-event-test.txt to s3://cyber-lab-data/cloudtrail-event-test.txt

# Delete the file (generates DeleteObject event)
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 rm s3://cyber-lab-data/cloudtrail-event-test.txt
# delete: s3://cyber-lab-data/cloudtrail-event-test.txt
```

### 4.2 Verify Log Delivery to S3

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 ls s3://cloudtrail-lab-logs/ \
    --recursive
# (no output — LocalStack does not fully simulate S3 log delivery)
```

### 4.3 Inspect Log Bucket Object Count

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api list-objects-v2 \
    --bucket cloudtrail-lab-logs \
    --max-keys 100
```

**Output:**
```json
{
    "IsTruncated": false,
    "Name": "cloudtrail-lab-logs",
    "Prefix": "",
    "MaxKeys": 100,
    "EncodingType": "url",
    "KeyCount": 0
}
```

### 4.4 Query CloudTrail Event History

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail lookup-events \
    --max-results 20
```

**Output:**
```json
{
    "Events": []
}
```

### 4.5 Create Additional Bucket to Trigger Events

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api create-bucket \
    --bucket cloudtrail-event-test-001
```

**Output:**
```json
{
    "Location": "/cloudtrail-event-test-001"
}
```

```bash
# Re-query events after bucket creation
aws --endpoint-url=http://127.0.0.1:4566 \
    cloudtrail lookup-events \
    --max-results 20
# { "Events": [] }
```

### 4.6 Final Bucket Inventory

```bash
aws --endpoint-url=http://127.0.0.1:4566 s3 ls
```

**Output:**
```
2026-09-19 00:03:07    cyber-lab-data
2026-09-19 00:20:58    cloudtrail-event-test-001
2026-09-19 00:15:17    cloudtrail-lab-logs
```

---


## Phase 5 — Public S3 Bucket Policy & Anonymous Access Testing

This phase explores S3 resource-based bucket policies by provisioning a **publicly readable bucket**, applying an open-access policy, and verifying unauthenticated object retrieval via both the AWS CLI and raw HTTP.

### 5.1 IAM Environment State Snapshot

Before proceeding, confirm current IAM state:

```bash
# Confirm IAM user exists
aws --endpoint-url=http://127.0.0.1:4566 \
    iam list-users
```

**Output:**
```json
{
    "Users": [
        {
            "Path": "/",
            "UserName": "cloud-lab-user",
            "UserId": "AIDA397WZ85ZZ86KAG8K",
            "Arn": "arn:aws:iam::000000000000:user/cloud-lab-user",
            "CreateDate": "2026-09-19T04:07:49.474741+00:00"
        }
    ]
}
```

```bash
# Confirm no IAM roles exist
aws --endpoint-url=http://127.0.0.1:4566 \
    iam list-roles
# { "Roles": [] }

# Confirm LabS3FullAccess policy is attached (AttachmentCount: 1)
aws --endpoint-url=http://127.0.0.1:4566 \
    iam list-policies \
    --scope Local
```

**Output:**
```json
{
    "Policies": [
        {
            "PolicyName": "LabS3FullAccess",
            "PolicyId": "ANPA7BK4F97M45RM0CF0",
            "Arn": "arn:aws:iam::000000000000:policy/LabS3FullAccess",
            "Path": "/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 1,
            "IsAttachable": true,
            "CreateDate": "2026-09-19T04:08:23.051160+00:00",
            "UpdateDate": "2026-09-19T04:08:23.051160+00:00"
        }
    ]
}
```

```bash
# Confirm cyber-lab-data still has no bucket policy
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-bucket-policy \
    --bucket cyber-lab-data
# aws: [ERROR]: NoSuchBucketPolicy
```

### 5.2 Create a Public-Facing S3 Bucket

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 mb s3://cyber-lab-public
```

**Output:**
```
make_bucket: cyber-lab-public
```

```bash
# Verify all buckets now present
aws --endpoint-url=http://127.0.0.1:4566 s3 ls
```

**Output:**
```
2026-09-19 00:03:07    cyber-lab-data
2026-09-19 00:20:58    cloudtrail-event-test-001
2026-09-19 00:15:17    cloudtrail-lab-logs
2026-09-19 00:33:58    cyber-lab-public
```

### 5.3 Upload a Test Object to the Public Bucket

```bash
echo "PUBLIC CLOUD SECURITY LAB TEST" > /tmp/public-test.txt

aws --endpoint-url=http://127.0.0.1:4566 \
    s3 cp /tmp/public-test.txt \
    s3://cyber-lab-public/
```

**Output:**
```
upload: ../../tmp/public-test.txt to s3://cyber-lab-public/public-test.txt
```

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3 ls s3://cyber-lab-public/
```

**Output:**
```
2026-09-19 00:34:20    31    public-test.txt
```

### 5.4 Define the Public Read Bucket Policy

Create `~/public-s3-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadLab",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cyber-lab-public/*"
    }
  ]
}
```

> [!WARNING]
> `"Principal": "*"` with no `Condition` block grants **unauthenticated public read** to every object in the bucket. This is the root cause of S3 data exposure incidents. Never apply this pattern to buckets containing sensitive data.

### 5.5 Apply the Bucket Policy

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api put-bucket-policy \
    --bucket cyber-lab-public \
    --policy file://$HOME/public-s3-policy.json
```

### 5.6 Verify Applied Policy

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-bucket-policy \
    --bucket cyber-lab-public \
    --query Policy \
    --output text
```

**Output:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadLab",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cyber-lab-public/*"
    }
  ]
}
```

### 5.7 Test Anonymous Access — AWS CLI (No Credentials)

Attempt to access the object with all AWS credential env vars stripped:

```bash
env -u AWS_ACCESS_KEY_ID \
env -u AWS_SECRET_ACCESS_KEY \
env -u AWS_SESSION_TOKEN \
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-object \
    --bucket cyber-lab-public \
    --key public-test.txt \
    /tmp/anonymous-public-test.txt
```

**Output:**
```
aws: [ERROR]: NoCredentials — Unable to locate credentials.
```

> [!NOTE]
> The AWS CLI **requires credentials** even for requests to publicly accessible resources. This is a CLI-level enforcement — the underlying S3 API itself does allow unauthenticated access via raw HTTP. Use `curl` to simulate a true anonymous client.

```bash
# Confirm file was NOT downloaded
cat /tmp/anonymous-public-test.txt
# cat: /tmp/anonymous-public-test.txt: No such file or directory
```

### 5.8 Test Anonymous Access — Raw HTTP via curl

Bypass the AWS CLI credential requirement by querying the S3 HTTP endpoint directly:

```bash
curl -i http://127.0.0.1:4566/cyber-lab-public/public-test.txt
```

**Output:**
```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Length: 31
Content-Type: text/plain
Date: Sat, 19 Sep 2026 04:36:49 GMT
ETag: "b76d995ab398a03da6f28b16246624a6"
Last-Modified: Sat, 19 Sep 2026 04:34:20 GMT
x-amz-id-2: 27591bf8-cc13-4f7e-87a1-0a9b02e04abb
x-amz-request-id: 27591bf8-cc13-4f7e-87a1-0a9b02e04abb
x-amz-storage-class: STANDARD

PUBLIC CLOUD SECURITY LAB TEST
```

**✅ Result:** Object retrieved with zero authentication — HTTP 200 with body confirmed.

### 5.9 Capture Headers and Object Separately

```bash
curl -sS \
  -D /tmp/public-s3-headers.txt \
  http://127.0.0.1:4566/cyber-lab-public/public-test.txt \
  -o /tmp/public-s3-object.txt

cat /tmp/public-s3-headers.txt
```

**Output:**
```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Length: 31
Content-Type: text/plain
Date: Sat, 19 Sep 2026 04:37:24 GMT
ETag: "b76d995ab398a03da6f28b16246624a6"
Last-Modified: Sat, 19 Sep 2026 04:34:20 GMT
x-amz-id-2: 8945d6d0-2819-45d7-b3ce-487fa911b69a
x-amz-request-id: 8945d6d0-2819-45d7-b3ce-487fa911b69a
x-amz-storage-class: STANDARD
```

```bash
cat /tmp/public-s3-object.txt
# PUBLIC CLOUD SECURITY LAB TEST
```

### 5.10 Test Authenticated Access — AWS CLI

```bash
aws --endpoint-url=http://127.0.0.1:4566 \
    s3api get-object \
    --bucket cyber-lab-public \
    --key public-test.txt \
    /tmp/authenticated-public-test.txt
```

**Output:**
```json
{
    "AcceptRanges": "bytes",
    "LastModified": "2026-09-19T04:34:20+00:00",
    "ContentLength": 31,
    "ETag": "\"b76d995ab398a03da6f28b16246624a6\"",
    "ChecksumCRC64NVME": "Hiq0mN7ZmbQ=",
    "ChecksumType": "FULL_OBJECT",
    "ContentType": "text/plain",
    "Metadata": {},
    "StorageClass": "STANDARD"
}
```

```bash
cat /tmp/authenticated-public-test.txt
# PUBLIC CLOUD SECURITY LAB TEST
```

**✅ Result:** Authenticated access also succeeds — policy allows both anonymous and credentialed reads.

### 5.11 Anonymous Bucket Listing via HTTP (List-Type=2)

```bash
curl -i "http://127.0.0.1:4566/cyber-lab-public?list-type=2"
```

**Output:**
```
HTTP/1.1 200 OK
content-length: 451
Content-Type: application/xml
Date: Sat, 19 Sep 2026 04:37:49 GMT
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Name>cyber-lab-public</Name>
  <Prefix></Prefix>
  <MaxKeys>1000</MaxKeys>
  <KeyCount>1</KeyCount>
  <IsTruncated>false</IsTruncated>
  <Contents>
    <Key>public-test.txt</Key>
    <LastModified>2026-09-19T04:34:20Z</LastModified>
    <ETag>&quot;b76d995ab398a03da6f28b16246624a6&quot;</ETag>
    <Size>31</Size>
    <StorageClass>STANDARD</StorageClass>
  </Contents>
</ListBucketResult>
```

> [!CAUTION]
> **Bucket listing is publicly accessible.** An attacker can enumerate all object keys in the bucket without credentials. In production, the bucket policy should include a `Deny` on `s3:ListBucket` for anonymous principals, and S3 Block Public Access should be enabled at the account level to prevent this class of misconfiguration entirely.

---

## Lab Findings & Observations

| # | Finding | Severity | Notes |
|---|---|---|---|
| 1 | **No bucket policy on `cyber-lab-data`** | 🔴 High | Object accessible to any authenticated identity with no resource-level control |
| 2 | **IAM policy grants `s3:*` on `Resource: *`** | 🔴 High | Overly permissive — violates least privilege |
| 3 | **CloudTrail trail created with logging disabled** | 🟠 Medium | Misconfiguration: trail exists but `IsLogging: false` by default |
| 4 | **Log file validation disabled** | 🟠 Medium | CloudTrail logs can be tampered with — no SHA-256 digest chain |
| 5 | **Multi-region trail not enabled** | 🟡 Low | Only `us-east-1` events captured; cross-region activity blind spots exist |
| 6 | **No CloudTrail Insights enabled** | 🟡 Low | Anomalous API rate detection not configured |
| 7 | **LocalStack `lookup-events` returns empty** | ℹ️ Info | Known limitation — LocalStack does not fully replicate CloudTrail event indexing |
| 8 | **`cyber-lab-public` bucket allows unauthenticated read** | 🔴 High | `Principal: *` with no conditions exposes all objects to anonymous HTTP access |
| 9 | **Bucket listing publicly accessible** | 🔴 High | No `Deny` on `s3:ListBucket` for anonymous principals — full object enumeration possible without credentials |
| 10 | **AWS CLI NoCredentials ≠ access denied** | ℹ️ Info | CLI refuses unsigned requests; raw HTTP confirms the policy grants genuine public access |

---

## Security Recommendations

### S3 Hardening

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyPublicAccess",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::cyber-lab-data",
        "arn:aws:s3:::cyber-lab-data/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

- Enable **S3 Block Public Access** at the account level
- Enable **Server-Side Encryption** (SSE-KMS) on sensitive buckets
- Enable **S3 Object Lock** for compliance/immutability requirements
- Enable **S3 Access Logging** to a separate audit bucket

### IAM Hardening

- Replace wildcard policies with **resource-scoped, action-specific** policies
- Enforce **MFA for all console and API access**
- Enable **IAM Access Analyzer** to identify unused permissions
- Rotate access keys every 90 days — use **AWS Secrets Manager** for rotation

### CloudTrail Hardening

```bash
# Enable with log validation and multi-region support
aws cloudtrail create-trail \
    --name production-trail \
    --s3-bucket-name cloudtrail-logs-prod \
    --is-multi-region-trail \
    --enable-log-file-validation \
    --include-global-service-events

# Start logging immediately
aws cloudtrail start-logging --name production-trail

# Enable CloudTrail Insights
aws cloudtrail put-insight-selectors \
    --trail-name production-trail \
    --insight-selectors '[{"InsightType":"ApiCallRateInsight"},{"InsightType":"ApiErrorRateInsight"}]'
```

---

## Key Takeaways

> [!TIP]
> **For Blue Teamers / Defenders:**
> - Always verify `IsLogging: true` after creating a CloudTrail trail — creation alone does **not** start logging
> - Monitor for `NoSuchBucketPolicy` errors in logs — they indicate exposed buckets with no access controls
> - Alert on `create-access-key` and `attach-user-policy` API calls — common privilege escalation signals

> [!TIP]
> **For Red Teamers / Assessors:**
> - Check CloudTrail status before performing actions — if `IsLogging: false`, activity may not be captured
> - Query `sts:GetCallerIdentity` to identify current privileges without leaving audit traces in most configurations
> - `s3api get-bucket-policy` returning `NoSuchBucketPolicy` identifies buckets with no resource-based controls

---

## Related Notes

- [[AWS IAM Privilege Escalation Techniques]]
- [[S3 Bucket Security Hardening Checklist]]
- [[CloudTrail Log Analysis with Athena]]
- [[LocalStack Setup & Configuration Guide]]

---

*Playbook maintained by the Security Architecture team. Review quarterly or after any major AWS environment change.*
