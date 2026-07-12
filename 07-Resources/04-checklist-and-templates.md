# Recon Checklist & Templates

## Pre-Scanning Checklist
- [ ] Scope confirmed (root domain, wildcards, excluded targets)
- [ ] Authorization confirmed (written scope, in-scope assets)
- [ ] Rate-limiting expectations reviewed
- [ ] Out-of-band collaborator ready (interactsh / Burp Collaborator)
- [ ] API keys configured (~/.config/subfinder/provider-config.yaml)
- [ ] Resolvers list downloaded and verified

## Recon Checklist
- [ ] Passive subdomain enumeration (subfinder + chaos + crt.sh)
- [ ] Certificate Transparency log analysis
- [ ] ASN -> IP range expansion
- [ ] DNS brute force (puredns / shuffledns)
- [ ] DNS zone transfer attempt
- [ ] Port scanning (naabu top-1000)
- [ ] HTTP probing (httpx: title, status, tech, CDN)
- [ ] Wayback URLs (waybackurls / gau)
- [ ] JavaScript file discovery + analysis
- [ ] Technology fingerprinting (httpx -tech-detect, wappalyzer)
- [ ] WAF detection (wafw00f)
- [ ] Content discovery (ffuf / feroxbuster)
- [ ] Subdomain takeover check (nuclei takeovers)
- [ ] Vulnerability scanning (nuclei CVEs + exposures)
- [ ] Visual inspection (screenshots or manual browse)

## Template: Quick Command Reference
```bash
# Passive + active pipeline
subfinder -d target.com -all -silent |   puredns resolve -r resolvers.txt |   naabu -top-ports 1000 -silent -json |   httpx -silent -title -status-code -tech-detect -o live.txt
```