---
title: Control - Secure Remote Access
category: Control
tags:
  - RemoteAccess
  - VPN
  - NetworkSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Secure Remote Access

## Control Objective

Ensure remote access is authenticated, authorized, monitored, and limited to business need.

## Risk Addressed

Unauthorized access, credential theft, exposed administration, remote compromise, excessive access.

## Control Type

Preventive. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Require MFA, device posture where appropriate, conditional access, least privilege, session logging, strong encryption, and rapid revocation.

## Evidence

VPN/remote access configuration, MFA policy, access groups, logs, device compliance records, exception approvals.

## Control Testing

Validate authorized users, MFA enforcement, access scope, logging, and leaver revocation.

## Failure Conditions

No MFA, unmanaged devices, broad access, shared accounts, stale access, missing logs.

## Metrics

Remote access MFA coverage, active remote users, remote access exceptions, failed/high-risk logins.

## Related Notes

- [[Network Security Domain]]
- [[Identity and Access Management]]
- [[Control - Multi-Factor Authentication]]

