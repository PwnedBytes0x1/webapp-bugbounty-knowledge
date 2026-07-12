# OWASP Top 10 (2021)

## A01: Broken Access Control
- IDOR, privilege escalation, force browsing
- Test: Compare responses as different users
- Fix: Server-side authorization on every request

## A02: Cryptographic Failures
- Weak TLS, exposed secrets, hardcoded keys
- Test: Check for HTTPS downgrade, weak ciphers
- Fix: Strong encryption, proper key management

## A03: Injection
- SQLi, NoSQLi, OS command injection, SSTI
- Test: Input validation bypass with special chars
- Fix: Parameterized queries, input validation

## A04: Insecure Design
- Business logic flaws, missing rate limits
- Test: Edge cases, race conditions, state machine
- Fix: Threat modeling in design phase

## A05: Security Misconfiguration
- Default creds, directory listing, debug modes
- Test: Scan for common misconfigurations
- Fix: Hardening guides, automation

## A06: Vulnerable Components
- Known CVEs in libraries, frameworks
- Test: Dependency scanning (OWASP Dependency Check)
- Fix: Regular updates, SBOM management

## A07: Auth Failures
- Weak passwords, credential stuffing, session bugs
- Test: Brute force, session fixation
- Fix: MFA, strong password policies

## A08: Data Integrity Failures
- Deserialization, insecure CI/CD
- Test: Serialized objects, signature verification
- Fix: Integrity checks, code signing

## A09: Logging Failures
- Missing audit trails, log injection
- Test: Check if attacks are logged
- Fix: Centralized logging, log analysis

## A10: SSRF
- Server-side request forgery
- Test: URL parameter manipulation
- Fix: URL validation, deny list