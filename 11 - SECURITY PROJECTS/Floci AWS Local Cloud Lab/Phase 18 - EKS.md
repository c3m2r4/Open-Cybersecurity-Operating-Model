# Phase 18: EKS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the EKS for the Floci Local Cloud Lab.

## ⚙️ Key Variables
- `\$EKS_IMAGE`
- `\$EKS_NAME`
- `\$EKS_STATUS`
- `\$EKS_SUPPORT_IMAGE`
- `\$LAB_ROLE_ARN`
- `\$SUBNET_ID`

## 💻 Implementation Script
```bash
# PHASE 18 — EKS

###############################################################################

  

info "PHASE 18 — EKS"

  

EKS_NAME="${LAB_PREFIX}-cluster"

EKS_STATUS=$(aws_local eks describe-cluster --name "$EKS_NAME" \
--query 'cluster.status' --output text 2>/dev/null)

  

if [[ "$EKS_STATUS" == "ACTIVE" || "$EKS_STATUS" == "CREATING" ]]; then

success "EKS cluster exists: $EKS_NAME"

elif [[ "$EKS_STATUS" == "FAILED" ]]; then

warn "Removing failed EKS cluster before retrying: $EKS_NAME"

if aws_provision eks delete-cluster --name "$EKS_NAME" >/dev/null 2>&1 \
&& wait_for_eks_deletion; then

EKS_STATUS=""

else

warn "EKS retry deferred because the failed cluster was not removed"

fi

fi

  

if [[ -z "$EKS_STATUS" || "$EKS_STATUS" == "None" ]]; then

if [[ -n "$SUBNET_ID" && "$SUBNET_ID" != "None" ]]; then

if ensure_docker_image "$EKS_IMAGE" \
&& ensure_docker_image "$EKS_SUPPORT_IMAGE"; then

aws_provision eks create-cluster \
--name "$EKS_NAME" \
--role-arn "$LAB_ROLE_ARN" \
--resources-vpc-config "subnetIds=$SUBNET_ID" \
>/dev/null 2>&1 \
&& success "Created EKS cluster: $EKS_NAME" \
|| warn "EKS cluster creation unavailable"

else

warn "EKS skipped because its image could not be pulled"

fi

else

warn "EKS skipped because no lab subnet is available"

fi

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
