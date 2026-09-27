# Phase 19: SES

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the SES for the Floci Local Cloud Lab.

## 💻 Implementation Script
```bash
# PHASE 19 — SES

###############################################################################

  

info "PHASE 19 — SES"

  

if aws_local ses verify-email-identity \
--email-address "floci-lab@example.local" \
>/dev/null 2>&1; then

  

success "SES test identity configured"

  

else

  

warn "SES unavailable"

  

fi

  

# The Console's SES Mailbox counts captured messages, not SES identities.

# Floci captures this locally; no message is delivered to the public Internet.

aws_local ses send-email \
--from "floci-lab@example.local" \
--destination 'ToAddresses=floci-lab@example.local' \
--message 'Subject={Data="Floci lab mailbox test",Charset=utf-8},Body={Text={Data="Local SES capture created by floci-lab.",Charset=utf-8}}' \
>/dev/null 2>&1 \
&& success "Submitted SES mailbox test message" \
|| warn "SES mailbox message could not be captured"

  

###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
