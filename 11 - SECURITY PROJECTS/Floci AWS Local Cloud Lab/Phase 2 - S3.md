# Phase 2: S3

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the S3 for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **S3**: Mb

## ⚙️ Key Variables
- `\$BUCKET`
- `\$S3_DATA`
- `\$S3_LOGS`
- `\$S3_PUBLIC`

## 💻 Implementation Script
```bash
# PHASE 2 — S3

###############################################################################

  

info "PHASE 2 — S3"

  

S3_DATA="${LAB_PREFIX}-data"

S3_PUBLIC="${LAB_PREFIX}-public"

S3_LOGS="${LAB_PREFIX}-logs"

  

for BUCKET in "$S3_DATA" "$S3_PUBLIC" "$S3_LOGS"; do

  

if aws_local s3api head-bucket --bucket "$BUCKET" >/dev/null 2>&1; then

success "S3 bucket exists: $BUCKET"

else

aws_local s3 mb "s3://$BUCKET" >/dev/null 2>&1 \
&& success "Created S3 bucket: $BUCKET" \
|| warn "Could not create S3 bucket: $BUCKET"

fi

  

done

  

echo "CONFIDENTIAL FLOCI CYBER LAB DATA" > /tmp/floci-secret.txt

  

aws_local s3 cp \
/tmp/floci-secret.txt \
"s3://$S3_DATA/cloud-secret.txt" \
>/dev/null 2>&1 \
&& success "Uploaded test object to $S3_DATA" \
|| warn "S3 upload failed"

  

###############################################################################

# PUBLIC S3 SECURITY TEST

###############################################################################

  

cat > /tmp/floci-public-policy.json <<JSON

{

"Version": "2012-10-17",

"Statement": [

{

"Sid": "PublicReadLab",

"Effect": "Allow",

"Principal": "*",

"Action": "s3:GetObject",

"Resource": "arn:aws:s3:::$S3_PUBLIC/*"

}

]

}

JSON

  

echo "PUBLIC CLOUD SECURITY LAB TEST" > /tmp/public-test.txt

  

aws_local s3 cp \
/tmp/public-test.txt \
"s3://$S3_PUBLIC/public-test.txt" \
>/dev/null 2>&1 \
|| true

  

aws_local s3api put-bucket-policy \
--bucket "$S3_PUBLIC" \
--policy file:///tmp/floci-public-policy.json \
>/dev/null 2>&1 \
|| warn "S3 public policy is not supported"

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
