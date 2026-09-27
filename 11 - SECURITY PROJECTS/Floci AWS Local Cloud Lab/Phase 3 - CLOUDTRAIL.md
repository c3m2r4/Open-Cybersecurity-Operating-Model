# Phase 3: CLOUDTRAIL

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the CLOUDTRAIL for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **CLOUDTRAIL**: Create Trail

## ⚙️ Key Variables
- `\$S3_LOGS`
- `\$TRAIL`

## 💻 Implementation Script
```bash
# PHASE 3 — CLOUDTRAIL

###############################################################################

  

info "PHASE 3 — CloudTrail"

  

TRAIL="${LAB_PREFIX}-trail"

  

if aws_local cloudtrail describe-trails \
--trail-name-list "$TRAIL" \
--query "trailList[0].Name" \
--output text 2>/dev/null | grep -q "$TRAIL"; then

  

success "CloudTrail trail exists: $TRAIL"

  

else

  

aws_local cloudtrail create-trail \
--name "$TRAIL" \
--s3-bucket-name "$S3_LOGS" \
>/dev/null 2>&1 \
&& success "Created CloudTrail trail" \
|| warn "CloudTrail trail creation unavailable"

  

fi

  

aws_local cloudtrail start-logging \
--name "$TRAIL" \
>/dev/null 2>&1 \
|| true

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
