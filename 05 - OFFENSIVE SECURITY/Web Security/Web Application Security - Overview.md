---
title: Web Application Security - Overview
category: Security Domain
tags:
  - WebSecurity
  - ApplicationSecurity
  - OffensiveSecurity
  - OWASP
date_created: 2026-08-24
status: MAINTAINED
---

# Web Application Security — Overview

## Purpose

This note covers the major web application security concepts from a risk-based perspective. For each area: what the threat is, what the risk is, how testing produces evidence, what the expected control is, how defenders can detect attacks, and how to remediate.

This is not a command cookbook. Detailed testing procedures reference OWASP and existing project notes.

> Reference: [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) | [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

## Domain Structure

| Area | Core Question |
|---|---|
| Authentication | Can the application correctly confirm who a user is? |
| Authorization | Can the application correctly control what a user is allowed to do? |
| Session Management | Can the application securely maintain authenticated state? |
| Input Validation | Does the application safely process all user-supplied input? |
| Injection | Can user input alter application behaviour or backend queries? |
| Access Control | Are object-level, function-level, and field-level permissions enforced? |
| Cryptography | Is sensitive data protected in transit and at rest? |
| Security Misconfiguration | Is the application, server, or infrastructure securely configured? |
| Business Logic | Are the application's intended workflows enforced, or can they be bypassed? |
| File Handling | Can users upload, access, or manipulate files in unintended ways? |
| API Security | Are API endpoints subject to the same controls as the web interface? |
| Secrets | Are credentials, tokens, and API keys protected? |
| Logging | Does the application log security-relevant events? |
| Monitoring | Are security events from the application visible to the security team? |

---

## Authentication

| Field | Detail |
|---|---|
| **Threat** | Unauthorized access through credential compromise, bypass, or weak authentication |
| **Risk** | Unauthorized access to user or administrative functions; data exposure; account takeover |
| **Test Focus** | MFA coverage; account lockout; password policy; credential transmission; brute force resistance; password reset flow; remember-me tokens; legacy protocol bypass |
| **Evidence** | Authentication request/response; MFA bypass demonstration; lockout behaviour; token analysis |
| **Expected Control** | MFA for sensitive functions; account lockout; secure credential transmission (HTTPS); phishing-resistant MFA for administrative access |
| **Detection** | Multiple failed authentication attempts; authentication from unusual location; authentication at unusual time; MFA fatigue events; new device authentication |
| **Remediation** | Enforce MFA; implement lockout and alerting; disable legacy authentication protocols; review password reset security |

---

## Authorization

| Field | Detail |
|---|---|
| **Threat** | Users accessing functions or data they are not permitted to access |
| **Risk** | Data exposure; privilege escalation; unauthorized business actions |
| **Test Focus** | Horizontal privilege escalation (accessing another user's data); vertical privilege escalation (accessing functions above assigned role); forced browsing (directly navigating to privileged URLs); parameter tampering to access restricted objects |
| **Evidence** | HTTP request/response showing unauthorized access; parameter manipulation demonstration |
| **Expected Control** | Server-side authorization checks on every request; role-based access control; deny-by-default |
| **Detection** | Access to resources not matching user role; high volume of 403/401 errors from a user; access to admin functions by non-admin accounts |
| **Remediation** | Implement server-side authorization for every function and object; do not rely on client-side controls alone |

> Authorization is the most common class of serious web application vulnerability. Client-side controls (hiding menu items, disabling buttons) are not authorization controls.

---

## Session Management

| Field | Detail |
|---|---|
| **Threat** | Session tokens stolen or forged to impersonate authenticated users |
| **Risk** | Account takeover; unauthorized access to authenticated sessions |
| **Test Focus** | Session token randomness and length; token transmission (cookie attributes); token invalidation on logout; session fixation; concurrent session handling |
| **Evidence** | Token capture; token prediction demonstration; cookie attribute analysis |
| **Expected Control** | Cryptographically random session tokens; Secure and HttpOnly cookie flags; token invalidation on logout; SameSite cookie policy |
| **Detection** | Simultaneous session use from different locations; impossible travel; session after account password change without re-authentication |
| **Remediation** | Generate cryptographically random tokens; enforce Secure/HttpOnly/SameSite; invalidate on logout and password change |

---

## Input Validation

| Field | Detail |
|---|---|
| **Threat** | User-supplied input processed without sanitization, enabling injection or unexpected behaviour |
| **Risk** | Injection attacks; data corruption; XSS; path traversal |
| **Test Focus** | All user-controlled inputs: query parameters, form fields, headers, file uploads, JSON/XML bodies; test for unexpected characters, encoding, and data types |
| **Evidence** | Input reflection or execution demonstration; error responses revealing processing logic |
| **Expected Control** | Server-side input validation; output encoding; parameterized queries; allow-list validation |
| **Detection** | Error spikes; unusual input patterns; WAF alerts |
| **Remediation** | Validate all input on the server; encode output; use parameterized queries; do not rely on client-side validation |

---

## Injection

| Field | Detail |
|---|---|
| **Threat** | User-controlled input alters the intended execution of backend queries or commands |
| **Risk** | Data exfiltration; authentication bypass; remote code execution; backend system compromise |
| **Test Focus** | SQL injection; command injection; LDAP injection; XML injection; template injection; SSRF; header injection |
| **Evidence** | Database error messages; data extraction demonstration; command execution evidence |
| **Expected Control** | Parameterized queries / prepared statements; ORM; input validation; least privilege for database accounts |
| **Detection** | Database error messages in application logs; anomalous query patterns; unusual outbound connections (for SSRF) |
| **Remediation** | Parameterized queries for all database interaction; avoid string concatenation in queries; restrict database account privileges |

---

## Access Control

| Field | Detail |
|---|---|
| **Threat** | Application fails to enforce access restrictions at the object, function, or field level |
| **Risk** | Users access data or functions beyond their authorization; mass data exposure |
| **Test Focus** | Insecure Direct Object References (IDOR); function-level access control; field-level access control; mass assignment |
| **Evidence** | HTTP request/response demonstrating access to another user's objects; mass assignment parameter modification |
| **Expected Control** | Object-level authorization checks; function-level authorization checks; allowlist of accepted fields |
| **Detection** | Access patterns deviating from normal user behaviour; access to IDs not associated with the user's session |
| **Remediation** | Implement object-level authorization for every data retrieval and modification; use indirect references where possible |

---

## Cryptography

| Field | Detail |
|---|---|
| **Threat** | Sensitive data exposed through weak or absent encryption |
| **Risk** | Credential exposure; PII exposure; session token theft; man-in-the-middle |
| **Test Focus** | TLS configuration; cipher suite strength; certificate validity; sensitive data transmission in cleartext; client-side storage of sensitive data; weak password hashing |
| **Evidence** | TLS configuration scan output; cleartext data capture; certificate analysis |
| **Expected Control** | TLS 1.2 minimum (TLS 1.3 preferred); strong cipher suites; HSTS; secure password hashing (bcrypt, Argon2) |
| **Detection** | Cleartext credential transmission; weak TLS negotiation; certificate expiry |
| **Remediation** | Enforce TLS 1.2+; disable weak cipher suites; implement HSTS; use strong hashing algorithms |

---

## Security Misconfiguration

| Field | Detail |
|---|---|
| **Threat** | Default, incomplete, or insecure configuration exposes functionality or data |
| **Risk** | Unauthorized access to administrative interfaces; information disclosure; increased attack surface |
| **Test Focus** | Default credentials; exposed admin interfaces; verbose error messages; directory listing; unnecessary features enabled; outdated software |
| **Evidence** | Default credential access; error message content; exposed admin path; directory listing |
| **Expected Control** | Secure baseline configuration; disable unnecessary features; custom error pages; restrict admin access by IP or network |
| **Detection** | Access to admin paths by non-admin accounts; scanner findings; configuration drift detection |
| **Remediation** | Establish and enforce secure baseline; review configuration before deployment; remove default accounts and credentials |

---

## Business Logic

| Field | Detail |
|---|---|
| **Threat** | Application workflow bypassed or abused to perform unintended actions |
| **Risk** | Financial manipulation; unauthorized transactions; discount/coupon abuse; process bypass |
| **Test Focus** | Workflow step skipping; parameter manipulation to alter business rules; race conditions; negative value inputs; repeated application of single-use features |
| **Evidence** | Demonstration of workflow bypass; manipulated transaction |
| **Expected Control** | Server-side workflow enforcement; business rule validation on server; idempotency checks |
| **Detection** | Transaction anomalies; unusual discount application rates; orders outside normal parameters |
| **Remediation** | Enforce business rules on the server; validate all business-relevant parameters server-side; implement rate limiting and anomaly detection |

---

## File Handling

| Field | Detail |
|---|---|
| **Threat** | Malicious file upload, path traversal, or unauthorized file access |
| **Risk** | Remote code execution via uploaded file; server file access; data disclosure |
| **Test Focus** | File upload type validation (content-type, extension, magic bytes); storage location; path traversal in file access parameters; file download authorization |
| **Evidence** | Uploaded malicious file served by application; path traversal file access |
| **Expected Control** | Allow-list file types by content inspection (not just extension); store uploads outside web root; validate file access authorization |
| **Detection** | Unusual file types uploaded; file access outside normal user scope; web shell execution indicators |
| **Remediation** | Validate file type by content, not name; store uploads outside web root; randomize file names; authorize every file access request |

---

## Secrets

| Field | Detail |
|---|---|
| **Threat** | Credentials, API keys, tokens, or secrets exposed in code, comments, headers, or client-side assets |
| **Risk** | Unauthorized access using exposed credentials; API key abuse; service account compromise |
| **Test Focus** | Source code review (JavaScript, HTML comments); response headers; error messages; exposed configuration files; git history |
| **Evidence** | Discovered credential or token; access using discovered secret |
| **Expected Control** | Secrets management system; secrets scanning in CI/CD; no hardcoded credentials; rotate exposed secrets immediately |
| **Detection** | Secrets scanning alerts; unusual API key usage patterns |
| **Remediation** | Remove secrets from code and configuration files; rotate any exposed secrets immediately; implement secrets management |

See: [[Control - Secrets Management]]

---

## Logging

| Field | Detail |
|---|---|
| **Threat** | Security-relevant events not logged, preventing detection and investigation |
| **Risk** | Undetected attacks; inability to investigate incidents; evidence gaps |
| **Test Focus** | Are authentication events logged? Are authorization failures logged? Are admin actions logged? Are errors logged with sufficient detail for investigation? |
| **Evidence** | Log review; demonstration of action not appearing in logs |
| **Expected Control** | Log authentication, authorization failures, admin actions, error events; log to centralized SIEM; protect log integrity |
| **Detection** | Gap in expected log events; log source failure |
| **Remediation** | Implement application-level logging for security events; forward to SIEM; protect log integrity |

See: [[SIEM - Overview]]

---

## Monitoring

| Field | Detail |
|---|---|
| **Threat** | Application security events not visible to security operations |
| **Risk** | Attacks proceed without detection; incident response delayed |
| **Test Focus** | Are application logs reaching SIEM? Do security events trigger alerts? Are authentication anomalies detected? |
| **Expected Control** | Application logs forwarded to SIEM; alert rules for authentication anomalies, authorization failures, error spikes |
| **Remediation** | Connect application logging to SIEM; implement use cases for web application events |

See: [[SOC - Operating Model]], [[Detection Engineering - Lifecycle]]

---

## Related Notes

- [[API Security - Overview]] — API-specific risks
- [[Vulnerability Assessment - Concepts]] — how to assess and prioritize web findings
- [[Risk-Based Offensive Security]] — how to rate web application findings
- [[Offensive Security Reporting]] — how to write web application findings professionally
- [[Control - Secure SDLC]] — how to prevent these issues in development
