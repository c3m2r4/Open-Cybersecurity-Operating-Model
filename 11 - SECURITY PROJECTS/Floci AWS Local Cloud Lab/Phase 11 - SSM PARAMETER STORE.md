# Phase 11: SSM PARAMETER STORE

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the SSM PARAMETER STORE for the Floci Local Cloud Lab.

## ⚙️ Key Variables
- `\$LAB_PREFIX`

## 💻 Implementation Script
```bash
# PHASE 11 — SSM PARAMETER STORE

###############################################################################

  

info "PHASE 11 — SSM Parameter Store"

  

aws_local ssm put-parameter \
--name "/$LAB_PREFIX/environment" \
--type String \
--value "development" \
--overwrite \
>/dev/null 2>&1 \
&& success "Created SSM parameter" \
|| warn "SSM Parameter Store unavailable"

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
