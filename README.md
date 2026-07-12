# Web Application Bug Bounty Knowledge Base

![GitHub last commit](https://img.shields.io/github/last-commit/PwnedBytes0x1/webapp-bugbounty-knowledge)
![GitHub repo size](https://img.shields.io/github/repo-size/PwnedBytes0x1/webapp-bugbounty-knowledge)
![GitHub stars](https://img.shields.io/github/stars/PwnedBytes0x1/webapp-bugbounty-knowledge?style=social)

> A comprehensive, 2026-edition knowledge base for web application bug bounty hunters.  
> Covers reconnaissance, vulnerability exploitation, tools, advanced topics, reporting, and resources.

---

## Sections

### 01 Fundamentals
| # | Topic | Description |
|---|-------|-------------|
| 1 | [HTTP Protocol](01-Fundamentals/01-http-protocol.md) | HTTP/1.1, HTTP/2, HTTP/3, request smuggling, desync attacks |
| 2 | [Web Technologies](01-Fundamentals/02-web-technologies.md) | Browser security model, SOP, CSP, browser fingerprinting |
| 3 | [OWASP Top 10](01-Fundamentals/03-owasp-top-10.md) | 2025 edition changes and bug bounty perspective |
| 4 | [Bug Bounty Essentials](01-Fundamentals/04-bug-bounty-essentials.md) | Platforms, payouts, methodology, time management |
| 5 | [Security Headers](01-Fundamentals/05-security-headers.md) | CSP, HSTS, CORS, COOP/COEP, Sec-Fetch-*, Permissions-Policy |

### 02 Reconnaissance
| # | Topic | Description |
|---|-------|-------------|
| 1 | [Passive Reconnaissance](02-Reconnaissance/01-passive-recon.md) | CT logs, ASN expansion, Shodan pivots, WHOIS, passive DNS |
| 2 | [Active Reconnaissance](02-Reconnaissance/02-active-recon.md) | Port scanning, naabu predictive scanning, evasion techniques |
| 3 | [Subdomain Enumeration](02-Reconnaissance/03-subdomain-enumeration.md) | 47+ sources, DNS brute force, NOERROR detection, permutations |
| 4 | [Content Discovery](02-Reconnaissance/04-content-discovery.md) | ffuf, feroxbuster, vhost fuzzing, backup files, .git exposure |
| 5 | [Technology Fingerprinting](02-Reconnaissance/05-technology-fingerprinting.md) | WAF detection (15+ vendors), JA3/JA4, bypass techniques |
| 6 | [JavaScript Analysis](02-Reconnaissance/06-js-analysis.md) | Source maps, secret detection (35+ patterns), endpoint extraction |
| 7 | [Port Scanning](02-Reconnaissance/07-port-scanning.md) | naabu deep dive, UDP probes, predictive scanning, service version detection |

### 03 Vulnerabilities
| # | Topic | Description |
|---|-------|-------------|
| 1 | [OWASP Top 10 2025](03-Vulnerabilities/01-owasp-top-10-2025.md) | Bug bounty signal by category |
| 2 | [Cross-Site Scripting (XSS)](03-Vulnerabilities/02-xss.md) | DOM clobbering (2026), mXSS, CSP bypass, Trusted Types |
| 3 | [SQL Injection](03-Vulnerabilities/03-sql-injection.md) | WAF bypass (LLM-based), blind, second-order, ORM injection |
| 4 | [NoSQL Injection](03-Vulnerabilities/04-nosql-injection.md) | MongoDB operator injection, $regex blind extraction, $where RCE |
| 5 | [SSRF](03-Vulnerabilities/05-ssrf.md) | Cloud metadata, IMDSv2 bypass, DNS rebinding, protocol smuggling |
| 6 | [IDOR](03-Vulnerabilities/06-idor.md) | UUID predictability, mass assignment, parameter pollution |
| 7 | [Auth Bypass](03-Vulnerabilities/07-authentication-bypass.md) | OAuth2, SAML, JWT attacks, MFA bypass, password reset |
| 8 | [Business Logic](03-Vulnerabilities/08-business-logic.md) | Race conditions, price manipulation, integer overflow |
| 9 | [API Security](03-Vulnerabilities/09-api-security.md) | GraphQL batching, mass assignment, rate limit bypass |
| 10 | [File Upload](03-Vulnerabilities/10-file-upload.md) | Extension bypass, polyglots, .htaccess, SVG XSS |
| 11 | [SSTI](03-Vulnerabilities/11-ssti.md) | Jinja2, Freemarker, Twig, Velocity, Smarty, blind SSTI |
| 12 | [XXE](03-Vulnerabilities/12-xxe.md) | Blind XXE, DTD smuggling, protocol smuggling |
| 13 | [Command Injection](03-Vulnerabilities/13-command-injection.md) | Blind, time-based, filter bypass, OOB detection |
| 14 | [CORS](03-Vulnerabilities/14-cors-misconfiguration.md) | Origin reflection, wildcard, null origin, preflight attacks |
| 15 | [Subdomain Takeover](03-Vulnerabilities/15-subdomain-takeover.md) | Service detection (15+), automation pipeline |
| 16 | [Prototype Pollution](03-Vulnerabilities/16-prototype-pollution.md) | Server-side RCE, NODE_OPTIONS, lodash merge, AST injection |
| 17 | [LLM & AI Security](03-Vulnerabilities/17-llm-ai-security.md) | OWASP LLM Top 10 v2, prompt injection, excessive agency |
| 18 | [Request Smuggling](03-Vulnerabilities/18-request-smuggling.md) | CL.TE, TE.CL, H2 downgrade, H3 QPACK desync |

### 04 Tools
| # | Topic | Description |
|---|-------|-------------|
| 1 | [Recon Tools](04-Tools/01-recon-tools.md) | subfinder, amass, httpx, interactsh, pipeline integration |
| 2 | [Scanning Tools](04-Tools/02-scanning-tools.md) | Nuclei, Katana, Uncover, scanner comparison |
| 3 | [Exploitation Tools](04-Tools/03-exploitation-tools.md) | Burp Suite, sqlmap, nosqli, commix, jwt_tool, Gopherus |
| 4 | [Automation Frameworks](04-Tools/04-automation-frameworks.md) | reconFTW, bbot, rengine, axiom, CI/CD integration |

### 05 Advanced Topics
| # | Topic | Description |
|---|-------|-------------|
| 1 | [Cloud Security](05-Advanced-Topics/01-cloud-security.md) | AWS/GCP/Azure attack surface, bucket enumeration |
| 2 | [Mobile Security](05-Advanced-Topics/02-mobile-security.md) | Android/iOS static/dynamic analysis, Frida, common findings |
| 3 | [Mobile Testing](05-Advanced-Topics/02-mobile-testing.md) | Setup pipeline, OWASP Mobile Top 10, deep link testing |
| 4 | [Web3 Security](05-Advanced-Topics/03-web3-security.md) | Smart contract vulns, DeFi/NFT attacks, testing tools |
| 5 | [GraphQL Security](05-Advanced-Topics/04-graphql-security.md) | Introspection, batching, DoS, defensive checklist |
| 6 | [JWT Attacks](05-Advanced-Topics/05-jwt-attacks.md) | Algorithm confusion, weak keys, KID injection, JWK injection |
| 7 | [JWT Security](05-Advanced-Topics/05-jwt-security.md) | Quick reference, critical checks, common CVEs |

### 06 Reporting
| # | Topic | Description |
|---|-------|-------------|
| 1 | [Report Template](06-Reporting/01-report-template.md) | Ready-to-use bug report template |
| 2 | [Writing Effective Reports](06-Reporting/01-writing-effective-reports.md) | Impact statements, replicable steps, common mistakes |
| 3 | [Screenshots & PoC](06-Reporting/02-screenshot-and-poc.md) | Best practices, PoC types (curl/HTML/video) |
| 4 | [Writing Guide](06-Reporting/02-writing-guide.md) | Title format, severity guidelines, triager perspective |
| 5 | [Collaboration & Remediation](06-Reporting/03-collaboration-and-remediation.md) | Triage, disclosure, fix verification |
| 6 | [Severity Matrix](06-Reporting/03-severity-matrix.md) | CVSS 3.1 vectors, bug bounty examples |
| 7 | [CVSS Scoring](06-Reporting/04-cvss-scoring.md) | Score ranges, bounty mapping |
| 8 | [Hall of Fame & Disclosure](06-Reporting/04-hall-of-fame-and-disclosure.md) | Disclosure types, timeline norms, legal protection |

### 07 Resources
| # | Topic | Description |
|---|-------|-------------|
| 1 | [Books & Courses](07-Resources/01-books-and-courses.md) | Recommended reading, free courses |
| 2 | [Books & Courses (Short)](07-Resources/01-books-courses.md) | Top recommendations quick list |
| 3 | [Communities & Platforms](07-Resources/02-communities-and-platforms.md) | Bug bounty platforms, Discord servers, writeup sources |
| 4 | [Labs & Platforms](07-Resources/02-labs-platforms.md) | Free/paid practice platforms |
| 5 | [Communities (Quick)](07-Resources/03-communities.md) | Active communities quick reference |
| 6 | [Wordlists & Tools](07-Resources/03-wordlists-and-tools-list.md) | SecLists, Assetnote, tool installation commands |
| 7 | [Checklist & Templates](07-Resources/04-checklist-and-templates.md) | Pre-scan checklist, recon template |
| 8 | [Methodology Cheatsheet](07-Resources/04-methodology-cheatsheet.md) | Core pipeline, quick commands by phase |

---

## Quick Start

```bash
# Clone the repo
git clone https://github.com/PwnedBytes0x1/webapp-bugbounty-knowledge.git
cd webapp-bugbounty-knowledge

# Start with Fundamentals → Recon → Vulnerabilities → Tools → Reports
```

## Recommended Reading Order

1. **Fundamentals** (HTTP, web tech, OWASP, methodology)
2. **Reconnaissance** (passive → active → subdomain → content → tech → JS)
3. **Vulnerabilities** (XSS → SQL → SSRF → IDOR → each type)
4. **Tools** (learn the tools you'll use while hunting)
5. **Advanced Topics** (cloud, mobile, Web3, GraphQL, JWT)
6. **Reporting** (how to write up your findings)
7. **Resources** (keep learning)

## License

MIT

---

*Maintained by [PwnedBytes0x1](https://github.com/PwnedBytes0x1). Contributions welcome!*
