# Phase 16: STEP FUNCTIONS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the STEP FUNCTIONS for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **STEPFUNCTIONS**: Create State Machine

## ⚙️ Key Variables
- `\$ACCOUNT_ID`
- `\$ROLE_ARN`
- `\$STATE_MACHINE_ARN`
- `\$STATE_MACHINE_FILE`

## 💻 Implementation Script
```bash
# PHASE 16 — STEP FUNCTIONS

###############################################################################

  

info "PHASE 16 — Step Functions"

  

STATE_MACHINE_FILE="/tmp/floci-state-machine.json"

  

cat > "$STATE_MACHINE_FILE" <<'JSON'

{

"Comment": "Floci Cybersecurity Lab Workflow",

"StartAt": "SecurityTest",

"States": {

"SecurityTest": {

"Type": "Pass",

"Result": {

"status": "success",

"environment": "floci-lab"

},

"End": true

}

}

}

JSON

  

ROLE_ARN="arn:aws:iam::$ACCOUNT_ID:role/floci-stepfunctions-role"

  

STATE_MACHINE_ARN=$(

aws_local stepfunctions list-state-machines \
--query "stateMachines[?name=='${LAB_PREFIX}-workflow'] | [0].stateMachineArn" \
--output text 2>/dev/null

)

  

if [[ -n "$STATE_MACHINE_ARN" && "$STATE_MACHINE_ARN" != "None" ]]; then

success "Step Functions state machine exists: $STATE_MACHINE_ARN"

else

aws_local stepfunctions create-state-machine \
--name "${LAB_PREFIX}-workflow" \
--definition file://"$STATE_MACHINE_FILE" \
--role-arn "$ROLE_ARN" \
>/dev/null 2>&1 \
&& success "Created Step Functions state machine" \
|| warn "Step Functions unavailable or requires a supported IAM role"

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
