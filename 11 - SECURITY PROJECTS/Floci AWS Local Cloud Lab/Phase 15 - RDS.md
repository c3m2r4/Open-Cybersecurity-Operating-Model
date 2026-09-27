# Phase 15: RDS

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the RDS for the Floci Local Cloud Lab.

## ⚙️ Key Variables
- `\$DB_IDENTIFIER`
- `\$RDS_POSTGRES_IMAGE`

## 💻 Implementation Script
```bash
# PHASE 15 — RDS

###############################################################################

  

info "PHASE 15 — RDS / DATABASE"

  

if aws_local rds describe-db-instances >/dev/null 2>&1; then

  

DB_IDENTIFIER="${LAB_PREFIX}-postgres"

  

if aws_local rds describe-db-instances \
--db-instance-identifier "$DB_IDENTIFIER" \
>/dev/null 2>&1; then

  

success "RDS instance exists: $DB_IDENTIFIER"

  

else

if ensure_docker_image "$RDS_POSTGRES_IMAGE"; then

aws_provision rds create-db-instance \
--db-instance-identifier "$DB_IDENTIFIER" \
--db-instance-class db.t3.micro \
--engine postgres \
--master-username labadmin \
--master-user-password 'LabPassword123!' \
--allocated-storage 20 \
>/dev/null 2>&1 \
&& success "Created RDS PostgreSQL instance" \
|| warn "RDS creation unavailable"

else

warn "RDS skipped because its image could not be pulled"

fi

  

fi

  

else

  

warn "RDS API unavailable"

  

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
