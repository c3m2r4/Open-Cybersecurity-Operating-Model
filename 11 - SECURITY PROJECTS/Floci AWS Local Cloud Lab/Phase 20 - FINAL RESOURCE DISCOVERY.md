# Phase 20: FINAL RESOURCE DISCOVERY

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the FINAL RESOURCE DISCOVERY for the Floci Local Cloud Lab.

## ⚙️ Key Variables
- `\$ACCOUNT_ID`
- `\$LOG_FILE`

## 💻 Implementation Script
```bash
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

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
