# Phase 4: VPC / NETWORKING

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the VPC / NETWORKING for the Floci Local Cloud Lab.

## 🏗️ Resources Provisioned
- **EC2**: Create Security Group
- **EC2**: Create Subnet
- **EC2**: Create Tags
- **EC2**: Create Vpc

## ⚙️ Key Variables
- `\$SG_ID`
- `\$SUBNET_2_ID`
- `\$SUBNET_ID`
- `\$VPC_ID`

## 💻 Implementation Script
```bash
# PHASE 4 — VPC / NETWORKING

###############################################################################

  

info "PHASE 4 — VPC / NETWORKING"

  

VPC_ID=""

  

VPC_ID=$(

aws_local ec2 describe-vpcs \
--filters "Name=tag:Name,Values=${LAB_PREFIX}-vpc" \
--query 'Vpcs[0].VpcId' \
--output text 2>/dev/null

)

  

if [[ "$VPC_ID" == "None" || -z "$VPC_ID" ]]; then

  

VPC_ID=$(

aws_local ec2 create-vpc \
--cidr-block 10.50.0.0/16 \
--query 'Vpc.VpcId' \
--output text 2>/dev/null

)

  

if [[ -n "$VPC_ID" && "$VPC_ID" != "None" ]]; then

aws_local ec2 create-tags \
--resources "$VPC_ID" \
--tags "Key=Name,Value=${LAB_PREFIX}-vpc" \
>/dev/null 2>&1 || true

  

success "Created VPC: $VPC_ID"

else

warn "VPC creation unavailable"

fi

  

else

  

success "VPC already exists: $VPC_ID"

  

fi

  

###############################################################################

# SUBNET

###############################################################################

  

SUBNET_ID=""

  

if [[ -n "$VPC_ID" && "$VPC_ID" != "None" ]]; then

  

SUBNET_ID=$(

aws_local ec2 describe-subnets \
--filters \
"Name=vpc-id,Values=$VPC_ID" \
"Name=tag:Name,Values=${LAB_PREFIX}-subnet" \
--query 'Subnets[0].SubnetId' \
--output text 2>/dev/null

)

  

if [[ "$SUBNET_ID" == "None" || -z "$SUBNET_ID" ]]; then

  

SUBNET_ID=$(

aws_local ec2 create-subnet \
--vpc-id "$VPC_ID" \
--cidr-block 10.50.1.0/24 \
--query 'Subnet.SubnetId' \
--output text 2>/dev/null

)

  

if [[ -n "$SUBNET_ID" && "$SUBNET_ID" != "None" ]]; then

aws_local ec2 create-tags \
--resources "$SUBNET_ID" \
--tags "Key=Name,Value=${LAB_PREFIX}-subnet" \
>/dev/null 2>&1 || true

  

success "Created subnet: $SUBNET_ID"

fi

  

else

  

success "Subnet exists: $SUBNET_ID"

  

fi

fi

  

###############################################################################

# SECOND SUBNET (used by services that require multiple availability zones)

###############################################################################

  

SUBNET_2_ID=""

  

if [[ -n "$VPC_ID" && "$VPC_ID" != "None" ]]; then

SUBNET_2_ID=$(aws_local ec2 describe-subnets \
--filters "Name=vpc-id,Values=$VPC_ID" "Name=tag:Name,Values=${LAB_PREFIX}-subnet-2" \
--query 'Subnets[0].SubnetId' --output text 2>/dev/null)

  

if [[ -z "$SUBNET_2_ID" || "$SUBNET_2_ID" == "None" ]]; then

SUBNET_2_ID=$(aws_local ec2 create-subnet --vpc-id "$VPC_ID" \
--cidr-block 10.50.2.0/24 --query 'Subnet.SubnetId' --output text 2>/dev/null)

if [[ -n "$SUBNET_2_ID" && "$SUBNET_2_ID" != "None" ]]; then

aws_local ec2 create-tags --resources "$SUBNET_2_ID" \
--tags "Key=Name,Value=${LAB_PREFIX}-subnet-2" >/dev/null 2>&1 || true

success "Created second subnet: $SUBNET_2_ID"

fi

fi

fi

  

###############################################################################

# SECURITY GROUP

###############################################################################

  

SG_ID=""

  

if [[ -n "$VPC_ID" && "$VPC_ID" != "None" ]]; then

  

SG_ID=$(

aws_local ec2 describe-security-groups \
--filters \
"Name=vpc-id,Values=$VPC_ID" \
"Name=group-name,Values=${LAB_PREFIX}-sg" \
--query 'SecurityGroups[0].GroupId' \
--output text 2>/dev/null

)

  

if [[ "$SG_ID" == "None" || -z "$SG_ID" ]]; then

  

SG_ID=$(

aws_local ec2 create-security-group \
--group-name "${LAB_PREFIX}-sg" \
--description "Floci Cybersecurity Lab Security Group" \
--vpc-id "$VPC_ID" \
--query 'GroupId' \
--output text 2>/dev/null

)

  

if [[ -n "$SG_ID" && "$SG_ID" != "None" ]]; then

  

# Lab-only ingress.

aws_local ec2 authorize-security-group-ingress \
--group-id "$SG_ID" \
--protocol tcp \
--port 22 \
--cidr 10.50.0.0/16 \
>/dev/null 2>&1 || true

  

aws_local ec2 authorize-security-group-ingress \
--group-id "$SG_ID" \
--protocol tcp \
--port 80 \
--cidr 10.50.0.0/16 \
>/dev/null 2>&1 || true

  

aws_local ec2 authorize-security-group-ingress \
--group-id "$SG_ID" \
--protocol tcp \
--port 443 \
--cidr 10.50.0.0/16 \
>/dev/null 2>&1 || true

  

success "Created security group: $SG_ID"

  

fi

  

else

  

success "Security group exists: $SG_ID"

  

fi

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
