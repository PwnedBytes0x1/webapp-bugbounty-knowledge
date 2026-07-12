# Reconnaissance Tools

## Subdomain Enumeration

### Passive
- **Subfinder**: Fast passive subdomain enumeration
  `subfinder -d target.com -o subs.txt`
- **Amass**: OWASP tool with passive + active modes
  `amass enum -d target.com`
- **Assetfinder**: Gather subdomains from various sources
  `assetfinder --subs-only target.com`
- **Sublist3r**: Aggregates from search engines
  `sublist3r -d target.com`
- **Crt.sh**: Certificate transparency logs
  `curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u`

### Active
- **Gobuster**: DNS bruteforce
  `gobuster dns -d target.com -w subdomains.txt`
- **dnsrecon**: DNS record enumeration
  `dnsrecon -d target.com -t brt -D subdomains.txt`

## Content Discovery

### Directory/File Bruteforce
- **Feroxbuster**: Fast recursive content discovery
  `feroxbuster -u https://target.com -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt`
- **ffuf**: Flexible fuzzer
  `ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt`
- **Dirsearch**: Web path scanner
  `dirsearch -u https://target.com`

## Technology Fingerprinting
- **wappalyzer**: Browser extension + CLI
- **WhatWeb**: Website profiling
  `whatweb https://target.com`
- **WafW00f**: WAF detection
  `wafw00f https://target.com`
- **BuiltWith**: Technology lookup

## Screenshot Collection
- **EyeWitness**: Automatically screenshots web services
  `eyewitness --web -f urls.txt`
- **Aquatone**: Screenshot + HTML report
  `cat subs.txt | aquatone`

## Network Scanning
- **masscan**: Fast port scanning
  `masscan -p1-65535 --rate=1000 target.com`
- **nmap**: Detailed service detection
  `nmap -sV -sC -p- target.com`