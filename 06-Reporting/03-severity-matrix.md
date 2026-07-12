# Bug Bounty Severity Matrix

## Critical (CVSS 9.0-10.0)
- Remote Code Execution
- SQL Injection (data extraction)
- Authentication Bypass (full)
- Server-Side Template Injection -> RCE
- Deserialization -> RCE
- Direct access to admin panel

## High (CVSS 7.0-8.9)
- XSS (stored with admin impact)
- SSRF (read cloud metadata)
- IDOR (read other users' data)
- File Upload (shell upload)
- CSRF (on sensitive actions)
- SQL Injection (blind/time-based)
- XXE (file read)
- Subdomain Takeover (cookie theft)
- Prototype Pollution (RCE)

## Medium (CVSS 4.0-6.9)
- XSS (reflected, no auth)
- CSRF (on non-sensitive actions)
- Open Redirect (with SSRF chain)
- IDOR (read non-sensitive data)
- CORS misconfiguration (no creds)
- Host header injection (password reset)
- Rate limiting bypass
- Missing security headers

## Low (CVSS 0.1-3.9)
- Information disclosure (non-sensitive)
- Missing security headers (no POC)
- CORS misconfiguration (no sensitive data)
- Clickjacking on non-sensitive pages
- Self-XSS
- Stack traces

## Informational
- Version disclosure
- Internal IP disclosure
- Directory listing (no sensitive files)
- Debug mode enabled