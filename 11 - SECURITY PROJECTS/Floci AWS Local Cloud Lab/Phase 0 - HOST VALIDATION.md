# Phase 0: HOST VALIDATION

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the HOST VALIDATION for the Floci Local Cloud Lab.

## 💻 Implementation Script
```bash
# PHASE 0 — HOST VALIDATION

###############################################################################

  

info "Checking required tools"

  

check_command aws

check_command curl

check_command timeout

  

success "Required tools available"

  

info "Testing Floci API"

  

if curl -fsS --max-time 10 "$ENDPOINT" >/tmp/floci-api-response 2>/dev/null; then

success "Floci endpoint is reachable: $ENDPOINT"

else

warn "Root endpoint did not return a successful HTTP response."

warn "Continuing because some local runtimes do not expose a root HTTP endpoint."

fi

  

info "Testing AWS STS"

  

if aws_local sts get-caller-identity >/tmp/floci-sts.json 2>/dev/null; then

success "AWS-compatible API is responding"

else

die "Floci AWS API is not reachable through $ENDPOINT"

fi

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
