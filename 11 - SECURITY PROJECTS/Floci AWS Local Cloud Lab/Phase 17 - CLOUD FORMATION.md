# Phase 17: CLOUD FORMATION

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the CLOUD FORMATION for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **CLOUDFORMATION**: Create Stack

## ⚙️ Key Variables
- `\$STACK_NAME`

## 💻 Implementation Script
```bash
# PHASE 17 — CLOUD FORMATION

###############################################################################

  

info "PHASE 17 — CloudFormation"

  

STACK_NAME="${LAB_PREFIX}-stack"

if aws_local cloudformation describe-stacks --stack-name "$STACK_NAME" >/dev/null 2>&1; then

success "CloudFormation stack exists: $STACK_NAME"

else

cat > /tmp/floci-cloudformation.json <<'JSON'

{

"AWSTemplateFormatVersion": "2010-09-09",

"Description": "Floci lab CloudFormation resource",

"Resources": {

"LabBucket": {"Type": "AWS::S3::Bucket"}

}

}

JSON

aws_local cloudformation create-stack \
--stack-name "$STACK_NAME" \
--template-body file:///tmp/floci-cloudformation.json \
>/dev/null 2>&1 \
&& success "Created CloudFormation stack: $STACK_NAME" \
|| warn "CloudFormation stack creation unavailable"

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
