---
title: Email Security Domain
category: Information Security Domain
tags:
  - EmailSecurity
  - Phishing
  - BEC
date_created: 2026-08-24
status: MAINTAINED
---

# Email Security Domain

## Executive Summary

Email security reduces risk from phishing, malware delivery, account compromise, business email compromise, executive impersonation, and data leakage through mail systems.

## Beginner

Email is one of the most common ways attackers reach people. Email security helps verify senders, block malicious content, and detect suspicious account activity.

## Practitioner

Core practices include SPF, DKIM, DMARC, anti-phishing protection, malware scanning, URL protection, attachment sandboxing, mailbox security, account compromise detection, executive impersonation protection, and secure mail configuration.

## Security Professional

Threats include credential phishing, malware attachments, malicious links, invoice fraud, OAuth consent abuse, mailbox rule abuse, lookalike domains, and executive impersonation.

## Controls

| Control Objective | Related Control |
|---|---|
| Verify email sender authenticity | [[Control - Email Authentication]] |
| Reduce phishing success | [[Control - Security Awareness Training]] |
| Detect compromised mailboxes | [[Control - Security Logging and SIEM]] |
| Protect accounts after credential theft | [[Control - Multi-Factor Authentication]] |
| Support reporting and response | [[Control - Incident Reporting]] |

## Evidence

Evidence includes SPF/DKIM/DMARC records, mail security configuration, phishing simulation results, reported phishing metrics, mailbox audit logs, suspicious inbox rule reports, email gateway logs, and incident tickets.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| DMARC policy state | KCI | Domain-specific - VERIFY |
| Phishing report rate | KPI / KCI | Better than training completion alone |
| Confirmed BEC incidents | KRI | Requires consistent incident classification |
| Malicious email click rate | KRI | Must be interpreted carefully |
| Mailbox audit coverage | KCI | Depends on platform capability |

## Management View

Email security protects payments, credentials, sensitive information, and executive trust. Management decisions may involve domain protection, user reporting programs, stronger authentication, or fraud process controls.

## Related Notes

- [[Identity and Access Management]]
- [[Security Awareness Domain]]
- [[Control - Email Authentication]]

