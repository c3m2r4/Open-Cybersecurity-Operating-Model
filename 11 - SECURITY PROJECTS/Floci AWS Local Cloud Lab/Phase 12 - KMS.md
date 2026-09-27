# Phase 12: KMS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the KMS for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **KMS**: Create Alias
- **KMS**: Create Key

## ⚙️ Key Variables
- `\$KMS_KEY_ID`

## 💻 Implementation Script
```bash
# PHASE 12 — KMS

###############################################################################

  

info "PHASE 12 — KMS"

  

KMS_KEY_ID=$(

aws_local kms describe-key \
--key-id "alias/${LAB_PREFIX}" \
--query 'KeyMetadata.KeyId' \
--output text 2>/dev/null

)

  

if [[ -z "$KMS_KEY_ID" || "$KMS_KEY_ID" == "None" ]]; then

KMS_KEY_ID=$(

aws_local kms create-key \
--description "Floci Cyber Lab KMS Key" \
--query 'KeyMetadata.KeyId' \
--output text 2>/dev/null

)

fi

  

if [[ -n "$KMS_KEY_ID" && "$KMS_KEY_ID" != "None" ]]; then

  

success "KMS key: $KMS_KEY_ID"

  

aws_local kms create-alias \
--alias-name "alias/${LAB_PREFIX}" \
--target-key-id "$KMS_KEY_ID" \
>/dev/null 2>&1 || true

  

else

  

warn "KMS unavailable"

  

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
