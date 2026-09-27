# Phase 1: ACCOUNT / IDENTITY

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the ACCOUNT / IDENTITY for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **IAM**: Create Policy
- **IAM**: Create Role
- **IAM**: Create User

## ⚙️ Key Variables
- `\$ACCOUNT_ID`
- `\$LAB_ROLE`
- `\$LAB_USER`
- `\$POLICY_NAME`

## 💻 Implementation Script
```bash
# PHASE 1 — ACCOUNT / IDENTITY

###############################################################################

  

info "PHASE 1 — IAM"

  

LAB_USER="${LAB_PREFIX}-user"

  

if aws_local iam get-user --user-name "$LAB_USER" >/dev/null 2>&1; then

success "IAM user already exists: $LAB_USER"

else

aws_local iam create-user \
--user-name "$LAB_USER" >/dev/null 2>&1 \
&& success "Created IAM user: $LAB_USER" \
|| warn "IAM user creation was not supported or failed"

fi

  

###############################################################################

# IAM POLICY

###############################################################################

  

cat > /tmp/floci-lab-policy.json <<'JSON'

{

"Version": "2012-10-17",

"Statement": [

{

"Sid": "FlociLabS3",

"Effect": "Allow",

"Action": [

"s3:*"

],

"Resource": "*"

},

{

"Sid": "FlociLabEC2",

"Effect": "Allow",

"Action": [

"ec2:*"

],

"Resource": "*"

},

{

"Sid": "FlociLabLogs",

"Effect": "Allow",

"Action": [

"logs:*"

],

"Resource": "*"

},

{

"Sid": "FlociLabMessaging",

"Effect": "Allow",

"Action": [

"sqs:*",

"sns:*"

],

"Resource": "*"

}

]

}

JSON

  

POLICY_NAME="${LAB_PREFIX}-administrator-lab"

LAB_ROLE="${LAB_PREFIX}-service-role"

LAB_ROLE_ARN="arn:aws:iam::$ACCOUNT_ID:role/$LAB_ROLE"

  

if aws_local iam get-policy \
--policy-arn "arn:aws:iam::$ACCOUNT_ID:policy/$POLICY_NAME" \
>/dev/null 2>&1; then

  

success "IAM policy already exists: $POLICY_NAME"

  

else

  

aws_local iam create-policy \
--policy-name "$POLICY_NAME" \
--policy-document file:///tmp/floci-lab-policy.json \
>/dev/null 2>&1 \
&& success "Created IAM policy: $POLICY_NAME" \
|| warn "IAM policy creation unavailable"

fi

  

aws_local iam attach-user-policy \
--user-name "$LAB_USER" \
--policy-arn "arn:aws:iam::$ACCOUNT_ID:policy/$POLICY_NAME" \
>/dev/null 2>&1 \
|| true

  

cat > /tmp/floci-lab-trust-policy.json <<'JSON'

{

"Version": "2012-10-17",

"Statement": [{

"Effect": "Allow",

"Principal": {"Service": ["lambda.amazonaws.com", "eks.amazonaws.com", "states.amazonaws.com"]},

"Action": "sts:AssumeRole"

}]

}

JSON

  

if ! aws_local iam get-role --role-name "$LAB_ROLE" >/dev/null 2>&1; then

aws_local iam create-role \
--role-name "$LAB_ROLE" \
--assume-role-policy-document file:///tmp/floci-lab-trust-policy.json \
>/dev/null 2>&1 \
&& success "Created service role: $LAB_ROLE" \
|| warn "Service role creation unavailable"

fi

  

aws_local iam attach-role-policy \
--role-name "$LAB_ROLE" \
--policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole" \
>/dev/null 2>&1 || true

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
