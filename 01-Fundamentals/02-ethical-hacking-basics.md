# Ethical Hacking for Bug Bounty: Operational Security

## Authorization Boundaries

Bug bounty authorization is narrow and revocable. Understand precisely what is in scope:
- **Explicit subdomains/endpoints only** — wildcard scopes (*.target.com) exclude third-party hosted content
- **Test accounts only** — never use real user data in PoCs
- **Rate limits** — violating rate limits during testing can get you banned
- **No social engineering** — phishing, physical access, and employee contact are always out of scope

## Operational Security (OPSEC)

### IP Management
- Use a VPS or VPN with a static IP for all testing. Residential IPs through datacenter proxies signal scanning activity
- Rotate IP addresses when rate-limited, but maintain a consistent testing identity for program communication
- Never scan from your home IP unless the program explicitly authorizes it

### Tooling Footprint
- Default tool configurations leave fingerprints. Modify User-Agent strings, request intervals, and concurrent connection counts
- Burp Suite's default CA certificate is fingerprintable — use a custom CA or CA certificate-less approaches for TLS tests
- Nuclei templates leave signature patterns that WAFs actively block

### Data Handling
- Screenshot only your own test accounts. Never capture another user's data in screenshots
- If you accidentally access PII, stop immediately, document what happened, and report it separately
- Delete any exfiltrated test data from your systems after the program resolves the report

### Communication Security
- Use the platform's messaging system (HackerOne/Bugcrowd/Immunefi). Do not email researchers directly
- Never disclose vulnerability details publicly until the program authorizes disclosure
- If a program goes unresponsive, follow the disclosure timeline in its policy — not public pressure

## Legal Reality

Bug bounty programs are not safe harbors. Even with a letter of authorization:
- You are still bound by the Computer Fraud and Abuse Act (CFAA) in the US and equivalent laws globally
- Scope changes mid-testing require re-confirmation
- Third-party services used by the target (Cloudflare, AWS, Auth0) are out of scope unless explicitly listed
- Rate limiting and DoS-adjacent testing can be prosecuted even when authorized
