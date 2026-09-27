# Setup & Configuration

Initial script configuration and helpers.

```bash
#!/usr/bin/env bash

  

###############################################################################

# Floci AWS Local Cloud Lab

#

# Purpose:

# Build a reusable AWS-style security/cloud laboratory against Floci.

#

# Host:

# Kali Linux / Linux

#

# Floci:

# AWS-compatible local runtime

#

# Console:

# http://127.0.0.1:4500/console/aws

#

# AWS API:

# http://127.0.0.1:4566

#

# IMPORTANT:

# This script is designed to be re-runnable.

# Existing resources are detected where possible instead of blindly failing.

###############################################################################

  

set -uo pipefail

  

###############################################################################

# CONFIGURATION

###############################################################################

  

export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID:-test}"

export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY:-test}"

export AWS_DEFAULT_REGION="${AWS_DEFAULT_REGION:-us-east-1}"

export AWS_PAGER=""

  

# AWS CLI endpoint used from the HOST.

# If your host cannot reach 127.0.0.1:4566, change this.

export ENDPOINT="${FLOCI_ENDPOINT:-http://127.0.0.1:4566}"

  

# Lab naming

LAB_PREFIX="cyber-lab"

  

ACCOUNT_ID="000000000000"

REGION="$AWS_DEFAULT_REGION"

  

# Keep unsupported or unhealthy local-service calls from making the bootstrap

# appear to hang. Override this for slower runtimes, for example:

# AWS_CLI_TIMEOUT=60 ./floci-lab.sh

AWS_CLI_TIMEOUT="${AWS_CLI_TIMEOUT:-20}"

# EKS and RDS start real local containers. They legitimately take longer than

# control-plane operations, so they use this separate, overridable bound.

AWS_PROVISION_TIMEOUT="${AWS_PROVISION_TIMEOUT:-180}"

AWS_IMAGE_PULL_TIMEOUT="${AWS_IMAGE_PULL_TIMEOUT:-300}"

EKS_IMAGE="${FLOCI_SERVICES_EKS_DEFAULT_IMAGE:-rancher/k3s:latest}"

# This Floci release resolves PostgreSQL to this pinned image internally.

RDS_POSTGRES_IMAGE="${FLOCI_SERVICES_RDS_DEFAULT_POSTGRES_IMAGE:-postgres:16.3-alpine}"

EKS_SUPPORT_IMAGE="${FLOCI_SERVICES_EKS_SUPPORT_IMAGE:-alpine/socat}"

# ---------------------------------------------------------------------------
# SSH / EC2 configuration
# Set SSH_PUBLIC_KEY to the path of your lab SSH public key.
# The matching private key is used to connect: ssh -i ~/.ssh/id_ed25519 ...
# ---------------------------------------------------------------------------
SSH_PUBLIC_KEY="${SSH_PUBLIC_KEY:-$HOME/.ssh/id_ed25519.pub}"
EC2_KEY_NAME="${EC2_KEY_NAME:-${LAB_PREFIX}-key}"
EC2_SSH_PORT=""   # populated after instance launch

  

###############################################################################

# OUTPUT

###############################################################################

  

LOG_DIR="/tmp/floci-lab"

LOG_FILE="$LOG_DIR/bootstrap.log"

  

mkdir -p "$LOG_DIR"

  

exec > >(tee -a "$LOG_FILE") 2>&1

  

echo

echo "=================================================================="

echo " FLOCI AWS LOCAL CLOUD LAB"

echo "=================================================================="

echo

echo "Endpoint : $ENDPOINT"

echo "Region : $REGION"

echo "Account : $ACCOUNT_ID"

echo "Log : $LOG_FILE"

echo

echo "=================================================================="

  

###############################################################################

# HELPERS

###############################################################################

  

die() {

echo

echo "[FATAL] $*"

echo

exit 1

}

  

info() {

echo

echo "[+] $*"

}

  

warn() {

echo

echo "[!] $*"

}

  

success() {

echo "[OK] $*"

}

  

run() {

"$@"

}

  

aws_local() {

timeout --foreground "$AWS_CLI_TIMEOUT" aws \
--endpoint-url="$ENDPOINT" \
--region="$REGION" \
--cli-connect-timeout 5 \
--cli-read-timeout "$AWS_CLI_TIMEOUT" \
--no-cli-pager \
"$@"

}

  

aws_provision() {

timeout --foreground "$AWS_PROVISION_TIMEOUT" aws \
--endpoint-url="$ENDPOINT" \
--region="$REGION" \
--cli-connect-timeout 5 \
--cli-read-timeout "$AWS_PROVISION_TIMEOUT" \
--no-cli-pager \
"$@"

}

  

ensure_docker_image() {

local image="$1"

  

if ! command -v docker >/dev/null 2>&1; then

warn "Docker is unavailable; Floci cannot provision $image"

return 1

fi

  

if docker image inspect "$image" >/dev/null 2>&1; then

return 0

fi

  

info "Pulling required image: $image"

timeout --foreground "$AWS_IMAGE_PULL_TIMEOUT" docker pull "$image"

}

  

wait_for_eks_deletion() {

local attempts=30

  

while (( attempts > 0 )); do

if ! aws_local eks describe-cluster --name "$EKS_NAME" >/dev/null 2>&1; then

return 0

fi

sleep 2

((attempts--))

done

  

warn "Timed out waiting for failed EKS cluster deletion"

return 1

}

  

check_command() {

command -v "$1" >/dev/null 2>&1 || die "$1 is not installed."

}

  

###############################################################################

```
