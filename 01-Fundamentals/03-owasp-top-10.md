# OWASP Top 10: 2025 Edition Deep Dive

## Overview - What Changed from 2021

OWASP analyzed 175,000+ CVEs mapped to 643 CWEs (up from 241 in 2021).

| Rank | Category | Change from 2021 |
|------|----------|------------------|
| A01 | Broken Access Control | Stayed #1, absorbed SSRF, expanded BOLA/BFLA |
| A02 | Security Misconfiguration | Surged from #5; 100% of apps had at least one |
| A03 | Software Supply Chain Failures | NEW - highest incidence rate (5.19%) |
| A04 | Cryptographic Failures | Dropped from #2 as TLS/HTTPS defaulted |
| A05 | Injection | Dropped from #3 as ORMs improved; XSS absorbed |
| A06 | Insecure Design | Unchanged |
| A07 | Authentication Failures | Renamed (was Identification and Auth Failures) |
| A08 | Software/Data Integrity Failures | Now covers CI/CD pipeline tampering |
| A09 | Security Logging and Alerting Failures | Renamed; 90-day SIEM retention recommended |
| A10 | Mishandling of Exceptional Conditions | NEW - community-voted |

SSRF moved into A01 (Broken Access Control).

## A01: Broken Access Control

Attack Vectors (2025-2026):
- IDOR: Manipulate numeric/UUID object references
- Horizontal Escalation: Same-role access to other users data
- Vertical Escalation: Regular user to admin endpoints
- Forced Browsing: Known URLs without UI links
- SSRF: Cloud metadata access as access control failure
- JWT Role Tampering: Modify claims client-side
- BOLA: Broken Object Level Authorization in APIs
- BFLA: Broken Function Level Authorization
- Multi-tenancy isolation failures in SaaS
- Microservice boundary IDOR chain

Detection: Replace object IDs, compare across roles, test all HTTP methods, GraphQL IDOR.

## A02: Security Misconfiguration (Surged #5 to #2)

100% of tested apps had at least one. Common: debug mode, default creds, CORS wildcard, directory listing, open cloud storage, verbose errors, missing headers, exposed .env/.git.

## A03: Software Supply Chain Failures (NEW)

Highest incidence rate (5.19%). Attacks: npm typosquatting, PyPI dependency confusion, CI/CD poisoning, maintainer takeover. Landmarks: XZ Utils backdoor (CVE-2024-3094), Polyfill.io supply chain attack. Mitigation: SBOM, pin deps, verify checksums, harden CI/CD.

## A05: Injection (Dropped #3 to #5)

SQLi still #1 CVE count. WAF bypass via comment injection, hex encoding. XSS absorbed: CSP bypass via nonce leakage, JSONP, disk cache reuse, DOM clobbering. NoSQL: $ne, $gt, $regex, $where. Prompt Injection: new for 2025-2026, test LLM-backed APIs with Burp.

## A10: Mishandling of Exceptional Conditions (NEW)

Community-voted addition. Fail-open auth, verbose error responses, resource leaks, rate limiting absent during errors. Test: disconnect auth service, send malformed input, trigger 500 errors.

## Free Tools: Semgrep (SAST), OWASP ZAP (DAST), Nuclei, Trivy, Gitleaks, ScoutSuite