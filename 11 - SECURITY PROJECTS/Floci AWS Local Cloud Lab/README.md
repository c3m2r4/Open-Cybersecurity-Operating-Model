# Floci AWS Local Cloud Lab

Welcome to the Floci AWS Local Cloud Lab! This directory contains the resources, scripts, and documentation necessary to provision a complete synthetic AWS environment locally for defensive security testing, monitoring, and infrastructure-as-code learning.

## 🚀 Quick Start: How to Run the Lab

The core of this lab is the `floci-lab.sh` script, which automates the provisioning of 20+ AWS resources (IAM, S3, VPC, EC2, Lambda, DynamoDB, etc.) against your local Floci instance.

### Prerequisites
Before running the script, ensure you have:
1. **Docker** installed and running on your host.
2. **Floci** running locally (usually via `docker-compose up` from the Floci root directory).
3. **AWS CLI v2** installed (`aws-cli`).

### Execution
To spin up the lab environment, open your terminal, navigate to this directory, and execute the script:

```bash
chmod +x floci-lab.sh
./floci-lab.sh
```

*Note: The script is designed to be idempotent. It will automatically check if resources already exist before attempting to create them, making it safe to re-run multiple times.*

## 📂 Documentation Structure

Because the provisioning script is over 2,300 lines long, we have broken it down into modular **Phase Notes** located in this folder. 

If you want to understand how a specific AWS service is configured (e.g., how the EKS cluster is deployed, or how the CloudTrail logging is attached to S3), simply read the corresponding Phase note (e.g., `Phase 18 - EKS.md`). Each note contains:
- A high-level overview.
- The specific resources provisioned.
- The isolated bash script segment used to provision that service.

Start your reading journey at the [[Master Guide]] for a complete table of contents!
