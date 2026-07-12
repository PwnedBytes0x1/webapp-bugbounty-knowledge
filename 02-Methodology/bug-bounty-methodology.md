# Bug Bounty Hunting Methodology

## Complete Workflow

SCOPE REVIEW -> RECONNAISSANCE -> DISCOVERY -> ENUMERATION -> VULNERABILITY TESTING -> PoC CREATION -> REPORTING

## Phase 0: Scope and Program Analysis

Before touching any tool:
1. Read scope rules carefully
2. Classify scope type: wide, medium, or narrow
3. Check for disclosed reports
4. Identify tech stack from job postings, GitHub, tech blogs

## Phase 1: Passive Reconnaissance

- Subdomain enumeration (certificate logs, search engines)
- ASN enumeration and IP range discovery
- GitHub code search for secrets and endpoints
- Shodan/Censys for exposed services
- Wayback Machine for historical endpoints

## Phase 2: Active Enumeration

- DNS brute force with wordlists
- Port scanning (Nmap, Masscan, Naabu)
- HTTP probing (httpx) for live hosts
- Technology fingerprinting
- JavaScript file collection and analysis
- Parameter discovery
- Directory/content fuzzing

## Phase 3: Vulnerability Discovery (Ebb and Flow)

The Ebb and Flow model:
1. Identify 3-5 attack vectors
2. Test briefly but thoroughly
3. Return to recon to expand surface
4. Repeat until no new findings

Priority order: IDOR -> Auth Bypass -> XSS -> SSRF -> SQLi -> Open Redirect -> CSRF -> Info Disclosure

## Phase 4: Exploitation and PoC

- Confirm the bug is reliably reproducible
- Chain low-severity bugs into critical impact
- Escalate severity (XSS -> session hijack -> ATO)
- Document every step

## Phase 5: Reporting

Structure:
1. Title - Clear, descriptive, includes impact
2. Severity - CVSS score and rating
3. Description - Technical explanation
4. Steps to Reproduce - Numbered, detailed
5. Impact - Business risk articulation
6. PoC - Screenshots, video, HTTP requests
7. Recommendations - Remediation guidance

## The 7-Question Gate (Before Reporting)

1. Can I reproduce it consistently?
2. Is it truly in scope?
3. Did I check for duplicates?
4. Can I escalate the severity?
5. Is the impact clearly demonstrated?
6. Is the report clear for non-technical readers?
7. Have I minimized the attack in my PoC?
