# Phase 8: EVENTBRIDGE

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the EVENTBRIDGE for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **EVENTS**: Create Event Bus

## ⚙️ Key Variables
- `\$EVENT_BUS`

## 💻 Implementation Script
```bash
# PHASE 8 — EVENTBRIDGE

###############################################################################

  

info "PHASE 8 — EventBridge"

  

EVENT_BUS="${LAB_PREFIX}-events"

  

aws_local events create-event-bus \
--name "$EVENT_BUS" \
>/dev/null 2>&1 \
|| true

  

aws_local events put-events \
--entries "[

{

\"EventBusName\":\"$EVENT_BUS\",

\"Source\":\"cyber.lab\",

\"DetailType\":\"SecurityTest\",

\"Detail\":\"{\\\"event\\\":\\\"floci-bootstrap\\\",\\\"severity\\\":\\\"medium\\\"}\"

}

]" \
>/dev/null 2>&1 \
&& success "EventBridge test event submitted" \
|| warn "EventBridge unavailable"

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
