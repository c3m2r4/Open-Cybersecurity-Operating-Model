# Phase 6: DYNAMODB

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the DYNAMODB for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **DYNAMODB**: Create Table

## ⚙️ Key Variables
- `\$TABLE`

## 💻 Implementation Script
```bash
# PHASE 6 — DYNAMODB

###############################################################################

  

info "PHASE 6 — DynamoDB"

  

TABLE="${LAB_PREFIX}-users"

  

if aws_local dynamodb describe-table \
--table-name "$TABLE" >/dev/null 2>&1; then

  

success "DynamoDB table exists: $TABLE"

  

else

  

aws_local dynamodb create-table \
--table-name "$TABLE" \
--attribute-definitions \
AttributeName=UserId,AttributeType=S \
--key-schema \
AttributeName=UserId,KeyType=HASH \
--billing-mode PAY_PER_REQUEST \
>/dev/null 2>&1 \
&& success "Created DynamoDB table: $TABLE" \
|| warn "DynamoDB is unavailable in this Floci runtime"

  

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
