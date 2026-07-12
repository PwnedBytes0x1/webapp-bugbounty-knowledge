# Complete Tool Reference

## Reconnaissance Tools

### Subdomain Enumeration
- **Subfinder** - Passive (30+ sources): subfinder -d target.com -all
- **Amass** - OWASP deep enumeration: amass enum -d target.com
- **Assetfinder** - Quick: assetfinder --subs-only target.com
- **Chaos** - ProjectDiscovery dataset

### DNS and Network
- **dnsx** - DNS resolution + CNAME check: cat subs.txt | dnsx -a -cname
- **puredns** - Wildcard-aware resolver
- **nmap** - Port scanning: nmap -sV -sC -p- target.com
- **naabu** - Fast port scan: naabu -host target.com -top-ports 1000
- **masscan** - Internet-scale: masscan -p1-65535 --rate=10000

### HTTP Probing
- **httpx** - HTTP probing + tech fingerprint
- **gowitness** - Screenshotting
- **aquatone** - Visual inspection

### Content Discovery
- **ffuf** - Fast web fuzzer
- **gobuster** - Directory + DNS brute force
- **dirsearch** - Recursive directory discovery
- **feroxbuster** - Rust-based recursive discovery
- **katana** - Next-gen crawler with JS rendering

## Vulnerability Scanning

- **Nuclei** - Template-based (9000+ templates)
- **Nikto** - Web server scanner
- **Nessus/OpenVAS** - Enterprise scanning

## Web Application Testing

- **Burp Suite Pro** - Industry standard
- **OWASP ZAP** - Free alternative

## Exploitation

- **SQLMap** - SQL injection automation
- **XSStrike** - XSS detection
- **Commix** - Command injection testing
- **Metasploit** - Exploitation framework
- **SSRFmap** - SSRF exploitation
- **JWT_Tool** - JWT attacks

## Automation

- **Interactsh** - OOB interaction detection
- **reconftw** - Full recon automation
- **recon0** - 9-stage Go pipeline
- **BugBounty Buddy** - AI-powered toolkit
- **JSPECTER** - JS intelligence + CVE correlation
