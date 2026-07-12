# OWASP Top 10:2025

The OWASP Top 10 is the industry-standard awareness document for web application security risks.

## The 2025 List

| Rank | Category | Key Change |
|------|----------|-----------|
| A01 | Broken Access Control | Now includes SSRF, BOLA, BFLA |
| A02 | Security Misconfiguration | Moved up from #5 |
| A03 | Software Supply Chain Failures | NEW - expanded |
| A04 | Cryptographic Failures | Dropped from #2 |
| A05 | Injection | Dropped from #3 |
| A06 | Insecure Design | Dropped from #4 |
| A07 | Authentication Failures | Holds #7 |
| A08 | Software/Data Integrity Failures | Holds #8 |
| A09 | Security Logging and Alerting | Renamed |
| A10 | Mishandling of Exceptional Conditions | NEW |

## A01: Broken Access Control
- 3.73% of tested applications affected
- Includes IDOR, privilege escalation, SSRF
- Manual testing only - scanners can't detect

## A02: Security Misconfiguration
- 100% of tested applications show some form
- Default credentials, open cloud storage, verbose errors
- Moved up from #5 in 2021

## A03: Software Supply Chain Failures
- 5.19% average incidence (highest)
- Dependencies, build systems, CI/CD pipelines
- Prevention: SBOM, dependency monitoring

## A05: Injection
- SQL, NoSQL, XSS, OS command, LDAP, template
- Prevention: Parameterized queries, input validation
