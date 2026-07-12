<div align="center">

# Web Application Bug Bounty Knowledge Base

[![GitHub last commit](https://img.shields.io/github/last-commit/PwnedBytes0x1/webapp-bugbounty-knowledge?style=for-the-badge&color=blue)](https://github.com/PwnedBytes0x1/webapp-bugbounty-knowledge/commits/main)
[![GitHub stars](https://img.shields.io/github/stars/PwnedBytes0x1/webapp-bugbounty-knowledge?style=for-the-badge&color=yellow)](https://github.com/PwnedBytes0x1/webapp-bugbounty-knowledge/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/PwnedBytes0x1/webapp-bugbounty-knowledge?style=for-the-badge&color=green)](https://github.com/PwnedBytes0x1/webapp-bugbounty-knowledge/network)
[![License: MIT](https://img.shields.io/badge/License-MIT-red?style=for-the-badge)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=for-the-badge)](CONTRIBUTING.md)

---

```
╔═══╗╔═══╗╔═══╗╔═══╗╔═══╗╔═══╗╔═══╗╔═══╗╔═══╗╔════╗╔═══╗╔═══╗
║╔═╗║║╔═╗║║╔═╗║║╔═╗║║╔══╝║╔══╝║╔═╗║║╔═╗║║╔═╗║║╔╗╔╗║║╔═╗║║╔═╗║
║╚═╝║║║║║║║╚═╝║║╚═╝║║╚══╗║╚══╗║║║║║║║║║║║║║║║║║║║║║╚═╝║║║║║║
║╔╗╔╝║║║║║║╔╗╔╝║╔╗╔╝║╔══╝║╔══╝║║║║║║║║║║║║║║║╚╝║║╚╝║╔╗╔╝║║║║║
║║║╚╗║╚═╝║║║║╚╗║║║╚╗║╚══╗║╚══╗║╚═╝║║╚═╝║║╚═╝║║║║║║║║║║╚╗║╚═╝║
╚╝╚═╝╚═══╝╚╝╚═╝╚╝╚═╝╚═══╝╚═══╝╚═══╝╚═══╝╚═══╝╚╝╚╝╚╝╚╝╚═╝╚═══╝
```

> **A comprehensive, community-driven knowledge base for web application bug bounty hunting.**
> Covers everything from foundational concepts to advanced exploitation techniques,
> with real-world examples, tool references, and best practices.

---

## Table of Contents

| # | Section | Description | Files |
|---|---------|-------------|-------|
| 01 | [Fundamentals](#-01-fundamentals) | Core concepts, ethics, mindset | 5 |
| 02 | [Reconnaissance](#-02-reconnaissance) | Subdomains, DNS, JS, cloud | 7 |
| 03 | [Vulnerabilities](#-03-vulnerabilities) | OWASP Top 10, XSS, SQLi, SSRF, API | 18 |
| 04 | [Tools](#-04-tools) | Scanners, frameworks, automation | 4 |
| 05 | [Advanced Topics](#-05-advanced-topics) | Cloud, mobile, Web3, GraphQL | 5 |
| 06 | [Reporting](#-06-reporting) | CVSS, PoC, triage | 4 |
| 07 | [Resources](#-07-resources) | Labs, books, wordlists | 4 |

---

## 📘 01 — Fundamentals

> Build a solid foundation in bug bounty hunting

| # | Topic |
|---|-------|
| 01 | [What is Bug Bounty?](01-Fundamentals/01-what-is-bug-bounty.md) |
| 02 | [Ethical Hacking Basics](01-Fundamentals/02-ethical-hacking-basics.md) |
| 03 | [Mindset & Methodology](01-Fundamentals/03-mindset-and-methodology.md) |
| 04 | [Legal & Ethics](01-Fundamentals/04-legal-and-ethics.md) |
| 05 | [Learning Path](01-Fundamentals/05-learning-path.md) |

---

## 🔎 02 — Reconnaissance

> Information gathering is 80% of bug hunting

| # | Topic |
|---|-------|
| 01 | [Subdomain Enumeration](02-Reconnaissance/01-subdomain-enumeration.md) |
| 02 | [DNS & Network Recon](02-Reconnaissance/02-dns-and-network-recon.md) |
| 03 | [URL & Content Discovery](02-Reconnaissance/03-url-and-content-discovery.md) |
| 04 | [JavaScript Analysis](02-Reconnaissance/04-javascript-analysis.md) |
| 05 | [GitHub Recon](02-Reconnaissance/05-github-recon.md) |
| 06 | [Cloud Asset Discovery](02-Reconnaissance/06-cloud-asset-discovery.md) |
| 07 | [Automation & One-liners](02-Reconnaissance/07-automation-and-oneliners.md) |

---

## 🛡️ 03 — Vulnerabilities

> Deep dives into 18 vulnerability classes with payloads, bypasses, and prevention

| # | Vulnerability | Severity |
|---|--------------|----------|
| 01 | [OWASP Top 10: 2025](03-Vulnerabilities/01-owasp-top-10-2025.md) | 🔴 Reference |
| 02 | [Cross-Site Scripting (XSS)](03-Vulnerabilities/02-xss.md) | 🟡 Medium-High |
| 03 | [SQL Injection](03-Vulnerabilities/03-sql-injection.md) | 🔴 Critical |
| 04 | [NoSQL Injection](03-Vulnerabilities/04-nosql-injection.md) | 🔴 Critical |
| 05 | [Server-Side Request Forgery (SSRF)](03-Vulnerabilities/05-ssrf.md) | 🔴 Critical |
| 06 | [Insecure Direct Object References (IDOR)](03-Vulnerabilities/06-idor.md) | 🟡 Medium-High |
| 07 | [Authentication Bypass](03-Vulnerabilities/07-authentication-bypass.md) | 🔴 Critical |
| 08 | [Business Logic Vulnerabilities](03-Vulnerabilities/08-business-logic.md) | 🟡 Medium-High |
| 09 | [API Security](03-Vulnerabilities/09-api-security.md) | 🔴 Critical |
| 10 | [File Upload Vulnerabilities](03-Vulnerabilities/10-file-upload.md) | 🔴 Critical |
| 11 | [Server-Side Template Injection (SSTI)](03-Vulnerabilities/11-ssti.md) | 🔴 Critical |
| 12 | [XML External Entities (XXE)](03-Vulnerabilities/12-xxe.md) | 🔴 Critical |
| 13 | [Command Injection](03-Vulnerabilities/13-command-injection.md) | 🔴 Critical |
| 14 | [CORS Misconfiguration](03-Vulnerabilities/14-cors-misconfiguration.md) | 🟡 Medium |
| 15 | [Subdomain Takeover](03-Vulnerabilities/15-subdomain-takeover.md) | 🟡 Medium-High |
| 16 | [Prototype Pollution](03-Vulnerabilities/16-prototype-pollution.md) | 🟡 Medium-High |
| 17 | [LLM / AI Security](03-Vulnerabilities/17-llm-ai-security.md) | 🟠 Emerging |
| 18 | [HTTP Request Smuggling](03-Vulnerabilities/18-request-smuggling.md) | 🔴 Critical |

---

## 🔧 04 — Tools

> Essential tools for every bug hunter

| # | Category |
|---|----------|
| 01 | [Reconnaissance Tools](04-Tools/01-recon-tools.md) |
| 02 | [Scanning Tools](04-Tools/02-scanning-tools.md) |
| 03 | [Exploitation Tools](04-Tools/03-exploitation-tools.md) |
| 04 | [Automation Frameworks](04-Tools/04-automation-frameworks.md) |

---

## 🚀 05 — Advanced Topics

> Specialized areas for experienced hunters

| # | Topic |
|---|-------|
| 01 | [Cloud Security (AWS/GCP/Azure)](05-Advanced-Topics/01-cloud-security.md) |
| 02 | [Mobile Application Testing](05-Advanced-Topics/02-mobile-testing.md) |
| 03 | [Web3 / Blockchain Security](05-Advanced-Topics/03-web3-security.md) |
| 04 | [GraphQL Security](05-Advanced-Topics/04-graphql-security.md) |
| 05 | [JWT Attacks](05-Advanced-Topics/05-jwt-attacks.md) |

---

## 📝 06 — Reporting

> Turn your findings into paid bounties

| # | Topic |
|---|-------|
| 01 | [Report Writing](06-Reporting/01-report-writing.md) |
| 02 | [CVSS Scoring](06-Reporting/02-cvss-scoring.md) |
| 03 | [Proof of Concept Creation](06-Reporting/03-poc-creation.md) |
| 04 | [Triage Interaction](06-Reporting/04-triage-interaction.md) |

---

## 📚 07 — Resources

> Curated learning materials and references

| # | Topic |
|---|-------|
| 01 | [Platforms & Programs](07-Resources/01-platforms.md) |
| 02 | [Labs & Courses](07-Resources/02-labs-and-courses.md) |
| 03 | [Wordlists & Payloads](07-Resources/03-wordlists-and-payloads.md) |
| 04 | [Books, Blogs & Communities](07-Resources/04-books-and-channels.md) |

---

## 📊 Repository Stats

| Metric | Value |
|--------|-------|
| Total Files | 48 markdown files |
| Sections | 7 |
| Vulnerabilities Covered | 18 |
| Total Topics | 44 |
| ⏱ Last Updated | 2026-07-12 |

## 🤝 Contributing

Contributions are welcome. Whether it's fixing a typo, adding a new vulnerability class, or improving an existing guide — every contribution helps the community.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built with ❤️ by [PwnedBytes0x1](https://github.com/PwnedBytes0x1)**

</div>
