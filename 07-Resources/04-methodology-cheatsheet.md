# Bug Bounty Methodology Cheatsheet

## Reconnaissance
```
1. Subdomain enumeration (Subfinder, Amass, Assetfinder)
2. Live probing (httpx)
3. Screenshot gathering (Aquatone)
4. Technology detection (WhatWeb, Wappalyzer)
5. Content discovery (Feroxbuster, ffuf)
6. Parameter discovery (Arjun, waybackurls)
7. JS file analysis (LinkFinder, JSParser)
8. Port scanning (masscan, naabu)
```

## Vulnerability Testing Flow
```
For each discovered endpoint/parameter:
1. Check access control (IDOR, privilege escalation)
2. Test input validation (XSS, SQLi, SSTI)
3. Test business logic (race conditions, state machine)
4. Test configuration (CORS, security headers, CSP)
5. Test authentication (JWT, OAuth, session management)
```

## Payload Quick Reference
```
SQLi: ' OR 1=1 --  |  " OR 1=1 --
XSS: <script>alert(1)</script>  |  <img src=x onerror=alert(1)>
SSTI: {{7*7}}  |  ${7*7}  |  <%= 7*7 %>
SSRF: http://127.0.0.1:8080  |  file:///etc/passwd
XXE: <!ENTITY xxe SYSTEM "file:///etc/passwd">
Prototype Pollution: {"__proto__":{"isAdmin":true}}
```

## Priority Matrix
| Priority | Action |
|----------|--------|
| Critical | RCE, SQLi (data exfil), Auth Bypass |
| High | XSS (stored), SSRF, IDOR (data) |
| Medium | XSS (reflected), CSRF, Open Redirect |
| Low | Info Disclosure, Missing Headers |