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

# PHASE 1 — ACCOUNT / IDENTITY

###############################################################################

  

info "PHASE 1 — IAM"

  

LAB_USER="${LAB_PREFIX}-user"

  

if aws_local iam get-user --user-name "$LAB_USER" >/dev/null 2>&1; then

success "IAM user already exists: $LAB_USER"

else

aws_local iam create-user \

--user-name "$LAB_USER" >/dev/null 2>&1 \

&& success "Created IAM user: $LAB_USER" \

|| warn "IAM user creation was not supported or failed"

fi

  

###############################################################################

# IAM POLICY

###############################################################################

  

cat > /tmp/floci-lab-policy.json <<'JSON'

{

"Version": "2012-10-17",

"Statement": [

{

"Sid": "FlociLabS3",

"Effect": "Allow",

"Action": [

"s3:*"

],

"Resource": "*"

},

{

"Sid": "FlociLabEC2",

"Effect": "Allow",

"Action": [

"ec2:*"

],

"Resource": "*"

},

{

"Sid": "FlociLabLogs",

"Effect": "Allow",

"Action": [

"logs:*"

],

"Resource": "*"

},

{

"Sid": "FlociLabMessaging",

"Effect": "Allow",

"Action": [

"sqs:*",

"sns:*"

],

"Resource": "*"

}

]

}

JSON

  

POLICY_NAME="${LAB_PREFIX}-administrator-lab"

LAB_ROLE="${LAB_PREFIX}-service-role"

LAB_ROLE_ARN="arn:aws:iam::$ACCOUNT_ID:role/$LAB_ROLE"

  

if aws_local iam get-policy \

--policy-arn "arn:aws:iam::$ACCOUNT_ID:policy/$POLICY_NAME" \

>/dev/null 2>&1; then

  

success "IAM policy already exists: $POLICY_NAME"

  

else

  

aws_local iam create-policy \

--policy-name "$POLICY_NAME" \

--policy-document file:///tmp/floci-lab-policy.json \

>/dev/null 2>&1 \

&& success "Created IAM policy: $POLICY_NAME" \

|| warn "IAM policy creation unavailable"

fi

  

aws_local iam attach-user-policy \

--user-name "$LAB_USER" \

--policy-arn "arn:aws:iam::$ACCOUNT_ID:policy/$POLICY_NAME" \

>/dev/null 2>&1 \

|| true

  

cat > /tmp/floci-lab-trust-policy.json <<'JSON'

{

"Version": "2012-10-17",

"Statement": [{

"Effect": "Allow",

"Principal": {"Service": ["lambda.amazonaws.com", "eks.amazonaws.com", "states.amazonaws.com"]},

"Action": "sts:AssumeRole"

}]

}

JSON

  

if ! aws_local iam get-role --role-name "$LAB_ROLE" >/dev/null 2>&1; then

aws_local iam create-role \

--role-name "$LAB_ROLE" \

--assume-role-policy-document file:///tmp/floci-lab-trust-policy.json \

>/dev/null 2>&1 \

&& success "Created service role: $LAB_ROLE" \

|| warn "Service role creation unavailable"

fi

  

aws_local iam attach-role-policy \

--role-name "$LAB_ROLE" \

--policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole" \

>/dev/null 2>&1 || true

  

###############################################################################

# PHASE 2 — S3

###############################################################################

  

info "PHASE 2 — S3"

  

S3_DATA="${LAB_PREFIX}-data"

S3_PUBLIC="${LAB_PREFIX}-public"

S3_LOGS="${LAB_PREFIX}-logs"

  

for BUCKET in "$S3_DATA" "$S3_PUBLIC" "$S3_LOGS"; do

  

if aws_local s3api head-bucket --bucket "$BUCKET" >/dev/null 2>&1; then

success "S3 bucket exists: $BUCKET"

else

aws_local s3 mb "s3://$BUCKET" >/dev/null 2>&1 \

&& success "Created S3 bucket: $BUCKET" \

|| warn "Could not create S3 bucket: $BUCKET"

fi

  

done

  

echo "CONFIDENTIAL FLOCI CYBER LAB DATA" > /tmp/floci-secret.txt

  

aws_local s3 cp \

/tmp/floci-secret.txt \

"s3://$S3_DATA/cloud-secret.txt" \

>/dev/null 2>&1 \

&& success "Uploaded test object to $S3_DATA" \

|| warn "S3 upload failed"

  

###############################################################################

# PUBLIC S3 SECURITY TEST

###############################################################################

  

cat > /tmp/floci-public-policy.json <<JSON

{

"Version": "2012-10-17",

"Statement": [

{

"Sid": "PublicReadLab",

"Effect": "Allow",

"Principal": "*",

"Action": "s3:GetObject",

"Resource": "arn:aws:s3:::$S3_PUBLIC/*"

}

]

}

JSON

  

echo "PUBLIC CLOUD SECURITY LAB TEST" > /tmp/public-test.txt

  

aws_local s3 cp \

/tmp/public-test.txt \

"s3://$S3_PUBLIC/public-test.txt" \

>/dev/null 2>&1 \

|| true

  

aws_local s3api put-bucket-policy \

--bucket "$S3_PUBLIC" \

--policy file:///tmp/floci-public-policy.json \

>/dev/null 2>&1 \

|| warn "S3 public policy is not supported"

  

###############################################################################

# PHASE 3 — CLOUDTRAIL

###############################################################################

  

info "PHASE 3 — CloudTrail"

  

TRAIL="${LAB_PREFIX}-trail"

  

if aws_local cloudtrail describe-trails \

--trail-name-list "$TRAIL" \

--query "trailList[0].Name" \

--output text 2>/dev/null | grep -q "$TRAIL"; then

  

success "CloudTrail trail exists: $TRAIL"

  

else

  

aws_local cloudtrail create-trail \

--name "$TRAIL" \

--s3-bucket-name "$S3_LOGS" \

>/dev/null 2>&1 \

&& success "Created CloudTrail trail" \

|| warn "CloudTrail trail creation unavailable"

  

fi

  

aws_local cloudtrail start-logging \

--name "$TRAIL" \

>/dev/null 2>&1 \

|| true

  

###############################################################################

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

# PHASE 5 — EC2

###############################################################################

  

info "PHASE 5 — EC2 / COMPUTE"

  

if aws_local ec2 describe-instances >/dev/null 2>&1; then

  

AMI_ID=$(

aws_local ec2 describe-images \

--owners self amazon \

--query 'Images[0].ImageId' \

--output text 2>/dev/null

)

  

if [[ "$AMI_ID" != "None" && -n "$AMI_ID" ]]; then

  

INSTANCE_ID=$(

aws_local ec2 describe-instances \

--filters "Name=tag:Name,Values=${LAB_PREFIX}-server" \

--query 'Reservations[].Instances[0].InstanceId' \

--output text 2>/dev/null

)

  

if [[ "$INSTANCE_ID" == "None" || -z "$INSTANCE_ID" ]]; then

  

if [[ -n "$SUBNET_ID" && -n "$SG_ID" ]]; then

  

INSTANCE_ID=$(

aws_local ec2 run-instances \

--image-id "$AMI_ID" \

--instance-type t3.micro \

--subnet-id "$SUBNET_ID" \

--security-group-ids "$SG_ID" \

--tag-specifications \

"ResourceType=instance,Tags=[{Key=Name,Value=${LAB_PREFIX}-server}]" \

--query 'Instances[0].InstanceId' \

--output text 2>/dev/null

)

  

[[ "$INSTANCE_ID" != "None" && -n "$INSTANCE_ID" ]] \

&& success "Created EC2 instance: $INSTANCE_ID" \

|| warn "EC2 instance creation failed"

  

fi

  

else

  

success "EC2 instance exists: $INSTANCE_ID"

  

fi

  

else

  

warn "No EC2 AMI is available in this Floci runtime"

  

fi

  

else

  

warn "EC2 API unavailable"

  

fi

  

###############################################################################

# PHASE 6 — DYNAMODB

###############################################################################

  

info "PHASE 6 — DynamoDB"

  

TABLE="${LAB_PREFIX}-users"

  

if aws_local dynamodb describe-table \

--table-name "$TABLE" >/dev/null 2>&1; then

  

success "DynamoDB table exists: $TABLE"

  

else

  

aws_local dynamodb create-table \

--table-name "$TABLE" \

--attribute-definitions \

AttributeName=UserId,AttributeType=S \

--key-schema \

AttributeName=UserId,KeyType=HASH \

--billing-mode PAY_PER_REQUEST \

>/dev/null 2>&1 \

&& success "Created DynamoDB table: $TABLE" \

|| warn "DynamoDB is unavailable in this Floci runtime"

  

fi

  

###############################################################################

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

# PHASE 7 — SQS

###############################################################################

  

info "PHASE 7 — SQS"

  

QUEUE_NAME="${LAB_PREFIX}-events"

  

QUEUE_URL=$(

aws_local sqs get-queue-url \

--queue-name "$QUEUE_NAME" \

--query QueueUrl \

--output text 2>/dev/null

)

  

if [[ -z "$QUEUE_URL" || "$QUEUE_URL" == "None" ]]; then

  

QUEUE_URL=$(

aws_local sqs create-queue \

--queue-name "$QUEUE_NAME" \

--query QueueUrl \

--output text 2>/dev/null

)

  

fi

  

if [[ -n "$QUEUE_URL" && "$QUEUE_URL" != "None" ]]; then

  

success "SQS queue: $QUEUE_URL"

  

aws_local sqs send-message \

--queue-url "$QUEUE_URL" \

--message-body '{"event":"floci-lab-test","severity":"medium"}' \

>/dev/null 2>&1 || true

  

else

  

warn "SQS unavailable"

  

fi

  

###############################################################################

# PHASE 8 — EVENTBRIDGE

###############################################################################

  

info "PHASE 8 — EventBridge"

  

EVENT_BUS="${LAB_PREFIX}-events"

  

aws_local events create-event-bus \

--name "$EVENT_BUS" \

>/dev/null 2>&1 \

|| true

  

aws_local events put-events \

--entries "[

{

\"EventBusName\":\"$EVENT_BUS\",

\"Source\":\"cyber.lab\",

\"DetailType\":\"SecurityTest\",

\"Detail\":\"{\\\"event\\\":\\\"floci-bootstrap\\\",\\\"severity\\\":\\\"medium\\\"}\"

}

]" \

>/dev/null 2>&1 \

&& success "EventBridge test event submitted" \

|| warn "EventBridge unavailable"

  

###############################################################################

# PHASE 9 — CLOUDWATCH LOGS

###############################################################################

  

info "PHASE 9 — CloudWatch Logs"

  

LOG_GROUP="/floci/$LAB_PREFIX"

  

aws_local logs create-log-group \

--log-group-name "$LOG_GROUP" \

>/dev/null 2>&1 \

|| true

  

aws_local logs create-log-stream \

--log-group-name "$LOG_GROUP" \

--log-stream-name "bootstrap" \

>/dev/null 2>&1 \

|| true

  

TIMESTAMP=$(date +%s000)

  

aws_local logs put-log-events \

--log-group-name "$LOG_GROUP" \

--log-stream-name "bootstrap" \

--log-events \

"timestamp=$TIMESTAMP,message=Floci cyber lab bootstrap completed" \

>/dev/null 2>&1 \

&& success "CloudWatch Logs configured" \

|| warn "CloudWatch Logs unavailable"

  

###############################################################################

# PHASE 10 — SECRETS MANAGER

###############################################################################

  

info "PHASE 10 — Secrets Manager"

  

SECRET_NAME="${LAB_PREFIX}/database"

  

if aws_local secretsmanager describe-secret \

--secret-id "$SECRET_NAME" >/dev/null 2>&1; then

  

success "Secret exists: $SECRET_NAME"

  

else

  

aws_local secretsmanager create-secret \

--name "$SECRET_NAME" \

--description "Floci cybersecurity lab test secret" \

--secret-string '{"username":"labuser","password":"LabPassword123!","environment":"lab"}' \

>/dev/null 2>&1 \

&& success "Created Secrets Manager secret" \

|| warn "Secrets Manager unavailable"

  

fi

  

###############################################################################

# PHASE 11 — SSM PARAMETER STORE

###############################################################################

  

info "PHASE 11 — SSM Parameter Store"

  

aws_local ssm put-parameter \

--name "/$LAB_PREFIX/environment" \

--type String \

--value "development" \

--overwrite \

>/dev/null 2>&1 \

&& success "Created SSM parameter" \

|| warn "SSM Parameter Store unavailable"

  

###############################################################################

# PHASE 12 — KMS

###############################################################################

  

info "PHASE 12 — KMS"

  

KMS_KEY_ID=$(

aws_local kms describe-key \

--key-id "alias/${LAB_PREFIX}" \

--query 'KeyMetadata.KeyId' \

--output text 2>/dev/null

)

  

if [[ -z "$KMS_KEY_ID" || "$KMS_KEY_ID" == "None" ]]; then

KMS_KEY_ID=$(

aws_local kms create-key \

--description "Floci Cyber Lab KMS Key" \

--query 'KeyMetadata.KeyId' \

--output text 2>/dev/null

)

fi

  

if [[ -n "$KMS_KEY_ID" && "$KMS_KEY_ID" != "None" ]]; then

  

success "KMS key: $KMS_KEY_ID"

  

aws_local kms create-alias \

--alias-name "alias/${LAB_PREFIX}" \

--target-key-id "$KMS_KEY_ID" \

>/dev/null 2>&1 || true

  

else

  

warn "KMS unavailable"

  

fi

  

###############################################################################

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

# PHASE 17 — CLOUD FORMATION

###############################################################################

  

info "PHASE 17 — CloudFormation"

  

STACK_NAME="${LAB_PREFIX}-stack"

if aws_local cloudformation describe-stacks --stack-name "$STACK_NAME" >/dev/null 2>&1; then

success "CloudFormation stack exists: $STACK_NAME"

else

cat > /tmp/floci-cloudformation.json <<'JSON'

{

"AWSTemplateFormatVersion": "2010-09-09",

"Description": "Floci lab CloudFormation resource",

"Resources": {

"LabBucket": {"Type": "AWS::S3::Bucket"}

}

}

JSON

aws_local cloudformation create-stack \

--stack-name "$STACK_NAME" \

--template-body file:///tmp/floci-cloudformation.json \

>/dev/null 2>&1 \

&& success "Created CloudFormation stack: $STACK_NAME" \

|| warn "CloudFormation stack creation unavailable"

fi

  

###############################################################################

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

# PHASE 20 — FINAL RESOURCE DISCOVERY

###############################################################################

  

info "PHASE 20 — FINAL RESOURCE INVENTORY"

  

echo

echo "---------------- S3 ----------------"

aws_local s3 ls 2>/dev/null || true

  

echo

echo "---------------- EC2 ----------------"

aws_local ec2 describe-instances \

--query 'Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType}' \

--output table 2>/dev/null || true

  

echo

echo "---------------- VPC ----------------"

aws_local ec2 describe-vpcs \

--query 'Vpcs[].{ID:VpcId,CIDR:CidrBlock}' \

--output table 2>/dev/null || true

  

echo

echo "---------------- DynamoDB ----------------"

aws_local dynamodb list-tables \

--output table 2>/dev/null || true

  

echo

echo "---------------- SQS ----------------"

aws_local sqs list-queues \

--output table 2>/dev/null || true

  

echo

echo "---------------- IAM ----------------"

aws_local iam list-users \

--query 'Users[].UserName' \

--output table 2>/dev/null || true

  

echo

echo "---------------- CloudWatch Logs ----------------"

aws_local logs describe-log-groups \

--query 'logGroups[].logGroupName' \

--output table 2>/dev/null || true

  

echo

echo "---------------- EventBridge ----------------"

aws_local events list-event-buses \

--query 'EventBuses[].Name' \

--output table 2>/dev/null || true

  

echo

echo "---------------- Lambda ----------------"

aws_local lambda list-functions \

--query 'Functions[].FunctionName' \

--output table 2>/dev/null || true

  

echo

echo "---------------- ELB ----------------"

aws_local elbv2 describe-load-balancers \

--query 'LoadBalancers[].LoadBalancerName' \

--output table 2>/dev/null || true

  

echo

echo "---------------- RDS ----------------"

aws_local rds describe-db-instances \

--query 'DBInstances[].DBInstanceIdentifier' \

--output table 2>/dev/null || true

  

echo

echo "---------------- EKS ----------------"

aws_local eks list-clusters --output table 2>/dev/null || true

  

echo

echo "---------------- CloudFormation ----------------"

aws_local cloudformation list-stacks \

--stack-status-filter CREATE_COMPLETE \

--query 'StackSummaries[].StackName' \

--output table 2>/dev/null || true

  

echo

echo "---------------- SES ----------------"

aws_local ses list-identities --output table 2>/dev/null || true

  

###############################################################################

# FINAL

###############################################################################

  

echo

echo "=================================================================="

echo " FLOCI LAB BOOTSTRAP COMPLETE"

echo "=================================================================="

echo

echo "Floci API : $ENDPOINT"

echo "Console : http://127.0.0.1:4500/console/aws"

echo "Region : $REGION"

echo "Account : $ACCOUNT_ID"

echo

echo "Log file : $LOG_FILE"

echo

echo "The script is safe to run again."

echo "Unsupported Floci services are reported instead of being fabricated."

echo

echo "=================================================================="

```