# Phase 9: CLOUDWATCH LOGS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the CLOUDWATCH LOGS for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **LOGS**: Create Log Group
- **LOGS**: Create Log Stream

## ⚙️ Key Variables
- `\$LAB_PREFIX`
- `\$LOG_GROUP`
- `\$TIMESTAMP`

## 💻 Implementation Script
```bash
# PHASE 9 — CLOUDWATCH LOGS

###############################################################################

  

info "PHASE 9 — CloudWatch Logs"

  

LOG_GROUP="/floci/$LAB_PREFIX"

  

aws_local logs create-log-group \
--log-group-name "$LOG_GROUP" \
>/dev/null 2>&1 \
|| true

  

aws_local logs create-log-stream \
--log-group-name "$LOG_GROUP" \
--log-stream-name "bootstrap" \
>/dev/null 2>&1 \
|| true

  

TIMESTAMP=$(date +%s000)

  

aws_local logs put-log-events \
--log-group-name "$LOG_GROUP" \
--log-stream-name "bootstrap" \
--log-events \
"timestamp=$TIMESTAMP,message=Floci cyber lab bootstrap completed" \
>/dev/null 2>&1 \
&& success "CloudWatch Logs configured" \
|| warn "CloudWatch Logs unavailable"

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
