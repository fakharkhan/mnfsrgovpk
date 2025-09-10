# MNFSR IT System – Security & Compliance Framework

## Document Information
- **Project**: Digital Transformation of MNFSR
- **Client**: Ministry of National Food Security & Research (MNFSR), Pakistan
- **Version**: 1.0
- **Date**: September 2025
- **Prepared By**: SOFT PYRAMID

---

## 1. Executive Summary
This framework defines the security and compliance controls for MNFSR’s six-module IT system. It aligns with Government of Pakistan IT security policies, PPRA Rules-2004 procurement compliance, and data protection obligations referenced in the RFP, implementing defense-in-depth, secure-by-design architecture, and continuous compliance monitoring.

---

## 2. Applicable Standards & Regulations
- Government of Pakistan IT security policies and data governance directives
- PPRA Rules-2004: procurement integrity, transparency, and Integrity Pact
- Data protection and privacy regulations (Pakistan) and RFP-referenced GDPR principles
- Accessibility: WCAG 2.1 AA for front-end experiences
- Information security management references (ISO/IEC 27001:2022) as implementation guidance

---

## 3. Security Architecture Overview
- Identity & Access: Central IAM with OpenID Connect, MFA, RBAC/ABAC; least privilege
- Network Security: WAF, API Gateway, private subnets, NSGs, bastion access, microservice mTLS
- Data Security: AES-256 at rest (DB, object storage), TLS 1.3 in transit, key vault with rotation
- Application Security: OWASP ASVS controls, input validation, CSRF/XSS mitigation, secure headers
- Logging & Monitoring: Immutable audit logs, SIEM ingestion, alerting, centralized metrics/traces

```mermaid
flowchart LR
  U[Users] --> WAF[WAF]
  WAF --> AGW[API Gateway]
  AGW --> SVC[Services]
  SVC --> SQL[(SQL Server TDE)]
  SVC --> OBJ[(Encrypted Object Storage)]
  SVC --> LOG[Audit/Logs -> SIEM]
  SVC --> IAM[Identity (OIDC, MFA, RBAC)]
```

---

## 4. Authentication & Authorization Framework
- MFA required for admins and sensitive operations
- RBAC roles: admin, manager, analyst, user; module-scoped permissions; front-end role/rights management per RFP (2.4.7.4)
- Privileged Access Management: time-bound elevation, approval workflow, session recording for admins
- Service-to-service authentication via client credentials and mTLS; audience-restricted JWTs

---

## 5. Data Protection & Privacy
- Data classification: Public, Internal, Restricted, Confidential (default: Internal)
- Minimization and purpose limitation for personal data; retention schedules per policy
- Encryption: AES-256 at rest; TLS 1.3; HSTS; secure cookies; PFS
- Pseudonymization/anonymization for analytics and test data
- DPIA templates for new high-risk processing (aligned with GDPR principles referenced in RFP)

---

## 6. Security Controls by Layer
- Perimeter: WAF rulesets, geo/IP allowlists for admin portals, DDoS protections
- App Layer: secure coding, dependency scanning, RASP/WA F integration, content security policy (CSP)
- Data Layer: TDE, cell/column encryption for PII, row-level security (RLS) for BI
- Endpoint/Admin: hardened jump hosts; MFA; logging; no direct DB access from user subnets
- Backups: encrypted at rest; tested restores; offsite copies

---

## 7. Audit Trails & Logging Requirements
- Immutable audit logs for: authentication, authorization changes, data exports, CRUD on sensitive entities
- Log format: structured JSON; include userId, role, ip, traceId, entity, action, before/after hashes
- Retention: 1 year online + 4 years archive (subject to policy); tamper-evident storage

---

## 8. Vulnerability Management
- SAST/DAST in CI/CD; dependency checks with CVE monitoring
- Quarterly vulnerability assessment; annual penetration testing; post-release targeted tests
- SLAs: Critical ≤ 7 days, High ≤ 14 days, Medium ≤ 30 days, Low in backlog

---

## 9. Incident Response (IR)
- Severity levels and playbooks (auth compromise, data leakage, service outage, malware)
- IR process: Detect → Triage → Contain → Eradicate → Recover → Post-mortem (RCA, actions)
- RACI: PPDU sponsor, MNFSR IT lead, vendor SOC, module owners
- Communications: internal/external notifications as per severity and legal requirements

---

## 10. Compliance Monitoring & Reporting
- Controls mapped to policies and evidence (e.g., MFA logs, backup reports, VAPT reports)
- Dashboards: security posture (patching, vuln backlog, auth anomalies), audit coverage, DR drill status
- Periodic reviews: monthly security reviews; quarterly management reporting

---

## 11. Secure Development Lifecycle (SSDLC)
- Requirements: threat modeling per module
- Design: security architecture review; data flow diagrams
- Implementation: secure coding checklists; peer reviews
- Verification: automated tests, ZAP scans, SAST/DAST gates
- Release: change approvals, rollout plan, rollback tested

---

## 12. References
- RFP: `docs/Planning_and_Development_7sep_Extracted_Text.txt`
- Architecture: `docs/Technical_Architecture_Document.md`
- Modules: `docs/Module_01_*` to `docs/Module_06_*`
