# Phase 14: ELB

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the ELB for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **ELBV2**: Create Load Balancer

## ⚙️ Key Variables
- `\$ELB_ARN`
- `\$ELB_NAME`
- `\$SUBNET_ID`

## 💻 Implementation Script
```bash
# PHASE 14 — ELB

###############################################################################

  

info "PHASE 14 — Elastic Load Balancing"

  

ELB_NAME="${LAB_PREFIX}-nlb"

ELB_ARN=$(aws_local elbv2 describe-load-balancers \
--names "$ELB_NAME" --query 'LoadBalancers[0].LoadBalancerArn' --output text 2>/dev/null)

  

if [[ -n "$ELB_ARN" && "$ELB_ARN" != "None" ]]; then

success "Load balancer exists: $ELB_NAME"

elif [[ -n "$SUBNET_ID" && "$SUBNET_ID" != "None" ]]; then

ELB_ARN=$(aws_local elbv2 create-load-balancer \
--name "$ELB_NAME" --type network --scheme internal --subnets "$SUBNET_ID" \
--query 'LoadBalancers[0].LoadBalancerArn' --output text 2>/dev/null)

[[ -n "$ELB_ARN" && "$ELB_ARN" != "None" ]] \
&& success "Created load balancer: $ELB_NAME" \
|| warn "ELB creation unavailable"

else

warn "ELB API unavailable"

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
