# Phase 10: SECRETS MANAGER

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the SECRETS MANAGER for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **SECRETSMANAGER**: Create Secret

## ⚙️ Key Variables
- `\$SECRET_NAME`

## 💻 Implementation Script
```bash
# PHASE 10 — SECRETS MANAGER

###############################################################################

  

info "PHASE 10 — Secrets Manager"

  

SECRET_NAME="${LAB_PREFIX}/database"

  

if aws_local secretsmanager describe-secret \
--secret-id "$SECRET_NAME" >/dev/null 2>&1; then

  

success "Secret exists: $SECRET_NAME"

  

else

  

aws_local secretsmanager create-secret \
--name "$SECRET_NAME" \
--description "Floci cybersecurity lab test secret" \
--secret-string '{"username":"labuser","password":"LabPassword123!","environment":"lab"}' \
>/dev/null 2>&1 \
&& success "Created Secrets Manager secret" \
|| warn "Secrets Manager unavailable"

  

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
