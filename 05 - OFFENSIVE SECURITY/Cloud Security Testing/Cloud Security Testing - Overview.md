---
title: Cloud Security Testing - Overview
category: Security Domain
tags:
  - CloudSecurity
  - OffensiveSecurity
  - IAM
date_created: 2026-08-24
status: MAINTAINED
---

# Cloud Security Testing — Overview

## Purpose

Cloud environments introduce a distinct security model from on-premises infrastructure. The shared responsibility model defines what the cloud provider secures and what the customer is responsible for. Cloud security testing focuses on what the customer is responsible for: identity, configuration, access, data, and logging.

> VERIFY: Cloud provider terms of service for security testing. Some providers require prior notification; check the specific provider policy before conducting security testing on cloud infrastructure.

See also: [[03 - INFORMATION SECURITY/Cloud Security]] for the defensive cloud security domain.

---

## Shared Responsibility Model

| Layer | Provider Responsibility | Customer Responsibility |
|---|---|---|
| Physical infrastructure | Provider | — |
| Hypervisor / compute | Provider | — |
| Network fabric | Provider | Customer network configuration |
| Operating system (IaaS) | — | Customer |
| Application (PaaS) | Provider (platform) | Customer (application) |
| Data | — | Customer |
| Identity and Access | Provider (platform IAM) | Customer (IAM configuration) |
| Logging and Monitoring | Provider (audit logs available) | Customer (must enable and consume) |

> Most cloud security weaknesses are in the customer's responsibility area — particularly IAM configuration, storage settings, and logging.

---

## Major Cloud Risk Areas

### Identity and Access Management (IAM)

| Risk | Description | Evidence |
|---|---|---|
| Overly permissive roles | IAM roles or policies grant more access than required | Role policy JSON showing wildcard permissions |
| Privilege escalation via IAM | IAM permissions that allow attaching more permissive roles to one's own identity | Permission boundary analysis; attaching role demonstration |
| Long-lived credentials | Access keys that are not rotated and not associated with a specific use case | Key creation date; last used date |
| External identity exposure | IAM roles that can be assumed by external accounts | Trust policy analysis showing external principal |
| MFA not enforced | Root account or privileged IAM users without MFA | IAM policy; credential report |

See: [[Control - Multi-Factor Authentication]]

---

### Storage and Data Exposure

| Risk | Description | Evidence |
|---|---|---|
| Public cloud storage | S3 buckets, Azure Blob containers, or GCP buckets accessible without authentication | Public access policy; unauthenticated access demonstration |
| Overly broad storage permissions | Storage accessible to all authenticated users in the account, not just intended services | Bucket policy analysis |
| Unencrypted storage | Sensitive data stored without encryption | Encryption configuration review |
| Versioning and logging disabled | No audit trail; no recovery capability | Bucket configuration review |

---

### Compute and Instance Security

| Risk | Description | Evidence |
|---|---|---|
| Instance metadata service (IMDS) exposure | IMDS accessible without IMDSv2 enforcement; SSRF can retrieve instance credentials | IMDS configuration; SSRF demonstration |
| Overpermissioned instance roles | EC2/compute instance has an IAM role with excessive permissions | Role policy review |
| Publicly exposed management ports | SSH (22) or RDP (3389) exposed to the internet | Security group review; port scan |
| Unpatched instances | Outdated operating systems or software on compute instances | OS version; scan output |

---

### Networking

| Risk | Description | Evidence |
|---|---|---|
| Security group misconfigurations | Inbound rules allow broad internet access to sensitive ports | Security group rule review |
| VPC peering without restriction | Peered networks with no effective segmentation | Route table and peering configuration review |
| Lack of network logging | VPC Flow Logs not enabled; no visibility into network traffic | Flow log configuration review |

---

### Logging and Monitoring

| Risk | Description | Evidence |
|---|---|---|
| Management plane audit logging disabled | CloudTrail / Azure Activity Log / GCP Audit Log not enabled or not covering all services | Audit log configuration review |
| Log retention insufficient | Logs deleted or retained for too short a period to support investigation | Retention policy review |
| Logs not forwarded to SIEM | Audit logs exist but are not being analyzed | SIEM data source inventory |
| No alerting on sensitive actions | No detection for root account use, IAM changes, or data access | Alert configuration review |

See: [[Control - Cloud Logging and Monitoring]]

---

### Container and Serverless Security

| Risk | Description | Evidence |
|---|---|---|
| Overpermissioned container roles | ECS task roles or pod service accounts with excessive IAM permissions | Permission review |
| Container escape | Host-mounted paths or privileged containers enable escape to host | Container configuration review |
| Serverless function environment exposure | Environment variables in Lambda/Azure Functions containing credentials | Function configuration review |
| Registry exposure | Container images in public registries or with sensitive data embedded | Registry access policy; image layer analysis |

---

## Cloud Testing Scope Considerations

| Consideration | Detail |
|---|---|
| **Authorization** | Cloud testing requires authorization from the account owner; some activities require cloud provider notification |
| **Multi-tenant risk** | Cloud infrastructure is shared at the physical and hypervisor level — exploitation that crosses tenant boundaries is strictly out of scope |
| **Logging** | All cloud testing activity will be visible in management plane audit logs |
| **Data handling** | Test in non-production environments where possible; if production access is required, confirm data handling requirements |
| **Cost impact** | Some testing activities can generate significant cloud costs — confirm resource creation/deletion is in scope |

---

## Detection in Cloud Environments

| Activity | Detection Source |
|---|---|
| IAM enumeration | CloudTrail: `ListUsers`, `ListRoles`, `GetPolicy` events at unusual volume |
| Privilege escalation via IAM | CloudTrail: `AttachRolePolicy`, `CreatePolicyVersion`, `UpdateAssumeRolePolicy` events |
| Access key use from unusual location | CloudTrail: API calls from unusual IP or geography |
| Root account use | CloudTrail: any root credential usage |
| Public storage access | S3 access logs: `GetObject` from unauthenticated principal |
| IMDS access | IMDSv1 requests without session token |

---

## Related Notes

- [[03 - INFORMATION SECURITY/Cloud Security]] — cloud defensive security domain
- [[Control - Cloud Logging and Monitoring]] — cloud logging control
- [[Risk-Based Offensive Security]] — how to assess cloud findings
- [[Offensive Security Safety and Authorization]] — authorization requirements for cloud testing
