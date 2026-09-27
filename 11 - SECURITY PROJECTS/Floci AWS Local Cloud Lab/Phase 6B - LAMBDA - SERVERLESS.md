# Phase 6B: LAMBDA / SERVERLESS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the LAMBDA / SERVERLESS for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **LAMBDA**: Create Function

## ⚙️ Key Variables
- `\$LAB_ROLE_ARN`
- `\$LAMBDA_DIR`
- `\$LAMBDA_NAME`

## 💻 Implementation Script
```bash
# PHASE 6B — LAMBDA / SERVERLESS

###############################################################################

  

info "PHASE 6B — Lambda / SERVERLESS"

  

LAMBDA_NAME="${LAB_PREFIX}-function"

  

if aws_local lambda get-function --function-name "$LAMBDA_NAME" >/dev/null 2>&1; then

success "Lambda function exists: $LAMBDA_NAME"

elif command -v zip >/dev/null 2>&1; then

LAMBDA_DIR="/tmp/${LAB_PREFIX}-lambda"

mkdir -p "$LAMBDA_DIR"

printf 'def handler(event, context):\n return {"statusCode": 200, "body": "Floci lab"}\n' > "$LAMBDA_DIR/lambda_function.py"

(cd "$LAMBDA_DIR" && zip -q -r function.zip lambda_function.py)

aws_local lambda create-function \
--function-name "$LAMBDA_NAME" \
--runtime python3.12 \
--role "$LAB_ROLE_ARN" \
--handler lambda_function.handler \
--zip-file "fileb://$LAMBDA_DIR/function.zip" \
>/dev/null 2>&1 \
&& success "Created Lambda function: $LAMBDA_NAME" \
|| warn "Lambda creation unavailable"

else

warn "Lambda skipped because zip is not installed"

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
