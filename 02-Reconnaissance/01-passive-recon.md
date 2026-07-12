# Passive Reconnaissance

## Certificate Transparency (crt.sh)
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u
```

## Search Engines
```
site:target.com intitle:"login"
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com "api_key" OR "secret"
site:target.com "bug bounty" OR "security"
```

## DNS Records (passive)
- SecurityTrails: Historical DNS data
- DNSDumpster: Visual DNS mapping
- VirusTotal: Passive DNS from malware analysis
- Shodan: Censys, ZoomEye for exposed services

## GitHub Dorking
```
org:target "password"
org:target "api_key"
org:target "aws_secret"
org:target ".env"
org:target "config.json"
org:target "-----BEGIN RSA PRIVATE KEY-----"
```

## Wayback Machine
```bash
waybackurls target.com | grep -E "\?|=" | uro > params.txt
gau target.com | uro > urls.txt
```

## Technology Stack (passive)
- BuiltWith: Full tech stack profile
- Wappalyzer: Browser extension
- WhatWeb: CLI tool
