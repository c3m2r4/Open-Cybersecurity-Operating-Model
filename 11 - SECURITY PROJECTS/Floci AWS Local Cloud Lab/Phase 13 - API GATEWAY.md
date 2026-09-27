# Phase 13: API GATEWAY

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the API GATEWAY for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **APIGATEWAY**: Create Rest Api

## ⚙️ Key Variables
- `\$API_ID`

## 💻 Implementation Script
```bash
# PHASE 13 — API GATEWAY

###############################################################################

  

info "PHASE 13 — API Gateway"

  

API_ID=$(

aws_local apigateway get-rest-apis \
--query "items[?name=='${LAB_PREFIX}-api'] | [0].id" \
--output text 2>/dev/null

)

  

if [[ -z "$API_ID" || "$API_ID" == "None" ]]; then

API_ID=$(

aws_local apigateway create-rest-api \
--name "${LAB_PREFIX}-api" \
--description "Floci Cybersecurity Lab API" \
--query id \
--output text 2>/dev/null

)

fi

  

if [[ -n "$API_ID" && "$API_ID" != "None" ]]; then

success "API Gateway REST API: $API_ID"

else

warn "API Gateway unavailable"

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
