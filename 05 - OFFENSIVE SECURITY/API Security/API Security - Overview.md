---
title: API Security - Overview
category: Security Domain
tags:
  - APISecurity
  - OffensiveSecurity
  - ApplicationSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# API Security — Overview

## Purpose

APIs are the primary interface for modern application communication. They are often subject to weaker security testing than web interfaces, exposed to broader audiences, and contain the same classes of vulnerabilities as web applications — plus several API-specific issues.

> Reference: [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)

---

## API Types

| Type | Description |
|---|---|
| **REST** | Resource-based; HTTP verbs; JSON payload; most common |
| **GraphQL** | Query language for APIs; flexible queries; specific attack surface (introspection, batching) |
| **SOAP** | XML-based; legacy enterprise environments; WSDL defines interface |
| **gRPC** | Binary protocol; Protocol Buffers; microservices environments |

---

## API-Specific Risk Areas

### Broken Object Level Authorization (BOLA / IDOR)

| Field | Detail |
|---|---|
| **Threat** | User modifies object identifiers in API requests to access other users' data |
| **Risk** | Mass data exposure; unauthorized data modification; account takeover via data manipulation |
| **Test Focus** | Replace numeric/UUID identifiers in API paths and parameters with another user's IDs; test whether authorization is enforced at the object level |
| **Evidence** | API response containing another user's data |
| **Expected Control** | Server-side object-level authorization on every request; indirect reference mapping |
| **Detection** | Access patterns where a user accesses IDs not in their expected dataset; high-volume sequential ID requests |
| **Remediation** | Implement object-level authorization for every API operation; validate that the requesting user is authorized to access the specific object |

> BOLA is consistently the highest-impact API vulnerability class. Many APIs enforce function-level authorization but not object-level authorization.

---

### Broken Authentication

| Field | Detail |
|---|---|
| **Threat** | API authentication bypassed or credentials compromised |
| **Risk** | Unauthorized API access; account takeover |
| **Test Focus** | Token strength and entropy; token expiry and revocation; credential brute force resistance; authentication for all endpoints including v1/legacy/internal |
| **Evidence** | Expired token accepted; token predictable; unauthenticated endpoint discovered |
| **Expected Control** | Strong token generation; token expiry; token revocation on logout; brute force protection |
| **Detection** | High-volume authentication failures; token use after logout; token use from unusual source |
| **Remediation** | Enforce authentication on all endpoints; implement token expiry and revocation; use established libraries for token generation |

---

### Excessive Data Exposure

| Field | Detail |
|---|---|
| **Threat** | API returns more data than the client needs, relying on the client to filter |
| **Risk** | Sensitive data exposure; credential exposure; PII disclosure |
| **Test Focus** | Compare API response to what the application displays; look for additional fields not shown in the UI (e.g., hashed passwords, internal IDs, sensitive attributes) |
| **Evidence** | API response containing sensitive fields not displayed in the UI |
| **Expected Control** | API response only returns fields necessary for the requesting function; server-side field filtering |
| **Detection** | Not easily detected reactively — prevention is the primary control |
| **Remediation** | Implement server-side response filtering; define explicit response schemas; do not return all object fields by default |

---

### Rate Limiting and Resource Exhaustion

| Field | Detail |
|---|---|
| **Threat** | Absence of rate limiting allows automated abuse, enumeration, or DoS |
| **Risk** | Credential brute force; user enumeration; data scraping; service unavailability |
| **Test Focus** | Test for rate limiting on authentication, password reset, OTP, data-retrieval endpoints; test for resource-intensive queries without limits |
| **Evidence** | Successful high-volume requests without throttling; OTP bypass via brute force |
| **Expected Control** | Rate limiting per user, per IP, and per endpoint; request size limits; pagination limits |
| **Detection** | High request rate from single source; authentication failure volume |
| **Remediation** | Implement rate limiting on all sensitive endpoints; implement pagination and query limits |

---

### Function Level Authorization

| Field | Detail |
|---|---|
| **Threat** | Administrative API functions accessible to standard users |
| **Risk** | Privilege escalation; unauthorized administrative actions |
| **Test Focus** | Map all API endpoints including administrative paths; test administrative endpoints with standard user tokens; test HTTP method variations |
| **Evidence** | Administrative function accessible with standard user token |
| **Expected Control** | Role-based authorization for every endpoint; deny by default |
| **Detection** | Standard user access to admin endpoint paths |
| **Remediation** | Implement function-level authorization; apply deny-by-default; document and review administrative API surface |

---

### Mass Assignment

| Field | Detail |
|---|---|
| **Threat** | Client-supplied JSON binds to object properties including ones not intended to be modifiable |
| **Risk** | Privilege escalation; unauthorized data modification |
| **Test Focus** | Add extra properties to API request bodies (e.g., `role`, `isAdmin`, `creditBalance`); test whether they are accepted |
| **Evidence** | API accepts unexpected property; privilege level changed via request |
| **Expected Control** | Explicit allow-list of accepted request properties; never bind client input directly to domain objects |
| **Detection** | Unexpected fields in API requests; privilege changes not performed through intended workflow |
| **Remediation** | Define explicit allowed properties per endpoint; reject unexpected properties |

---

### Security Misconfiguration

| Field | Detail |
|---|---|
| **Threat** | Default or insecure API configuration exposes functionality or information |
| **Risk** | Unauthorized access; information disclosure; increased attack surface |
| **Test Focus** | GraphQL introspection enabled in production; CORS misconfiguration; verbose error messages; debug endpoints; old API versions active |
| **Evidence** | Introspection query returns schema; CORS allows arbitrary origin; debug endpoint accessible |
| **Expected Control** | Disable introspection in production; restrict CORS to known origins; custom error responses; version management |
| **Detection** | Introspection queries from external sources; unusual CORS origin use |
| **Remediation** | Disable debug features in production; restrict CORS; implement version lifecycle management |

---

## API Inventory

A prerequisite for API security testing is knowing what APIs exist:

- Internal APIs used by the application's own frontend
- Partner APIs exposed to external systems
- Legacy API versions still active
- Undocumented or shadow APIs
- Third-party APIs consumed by the application

> Undocumented APIs are not protected APIs. They are often the most vulnerable because they were built without security review.

---

## Detection for API Attacks

| Attack | Detection Signal |
|---|---|
| BOLA | Unusual ID access patterns; sequential ID enumeration |
| Credential brute force | High authentication failure volume; lockout events |
| Token abuse | Token use after expected expiry; token from unusual IP |
| Excessive data retrieval | High-volume API requests; large response payloads |
| Function-level escalation | Admin path access by non-admin accounts |
| Mass assignment | Unexpected fields in request payloads |

Route API logs to [[SIEM - Overview]] for correlation and alerting.

---

## Related Notes

- [[Web Application Security - Overview]] — web context for API security
- [[Risk-Based Offensive Security]] — how to assess API findings
- [[Offensive Security Reporting]] — how to document API findings
- [[Control - Secure SDLC]] — how to prevent API vulnerabilities in development
