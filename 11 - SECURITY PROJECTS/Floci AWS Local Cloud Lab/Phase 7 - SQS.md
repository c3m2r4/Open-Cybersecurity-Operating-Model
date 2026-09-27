# Phase 7: SQS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the SQS for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **SQS**: Create Queue

## ⚙️ Key Variables
- `\$QUEUE_NAME`
- `\$QUEUE_URL`

## 💻 Implementation Script
```bash
# PHASE 7 — SQS

###############################################################################

  

info "PHASE 7 — SQS"

  

QUEUE_NAME="${LAB_PREFIX}-events"

  

QUEUE_URL=$(

aws_local sqs get-queue-url \
--queue-name "$QUEUE_NAME" \
--query QueueUrl \
--output text 2>/dev/null

)

  

if [[ -z "$QUEUE_URL" || "$QUEUE_URL" == "None" ]]; then

  

QUEUE_URL=$(

aws_local sqs create-queue \
--queue-name "$QUEUE_NAME" \
--query QueueUrl \
--output text 2>/dev/null

)

  

fi

  

if [[ -n "$QUEUE_URL" && "$QUEUE_URL" != "None" ]]; then

  

success "SQS queue: $QUEUE_URL"

  

aws_local sqs send-message \
--queue-url "$QUEUE_URL" \
--message-body '{"event":"floci-lab-test","severity":"medium"}' \
>/dev/null 2>&1 || true

  

else

  

warn "SQS unavailable"

  

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
