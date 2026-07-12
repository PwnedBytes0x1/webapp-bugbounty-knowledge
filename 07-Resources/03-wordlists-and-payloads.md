# Wordlists & Payloads

## Essential Wordlists

### SecLists (by Daniel Miessler)
```
# Install
git clone https://github.com/danielmiessler/SecLists.git

# Key files:
Discovery/DNS/subdomains-top1million-20000.txt
Discovery/Web-Content/raft-small-words.txt
Discovery/Web-Content/Common-DB-Backups.txt
Discovery/Web-Content/big.txt
Discovery/Web-Content/directory-list-2.3-medium.txt
Discovery/Web-Content/api/api-endpoints.txt
Fuzzing/parameters.txt
Payloads/XSS/xss-payload-list.txt
Payloads/SQLi/SQLi.txt
Payloads/SSRF/ssrf.txt
```

### Other Wordlists

| Wordlist | Description |
|----------|-------------|
| **commonspeak2** | URL-based wordlists from common crawl data |
| **JHaddix-all.txt** | Comprehensive subdomain wordlist |
| **Assetnote Wordlists** | Based on actual web crawl data |
| **Raft Wordlists** | Small, curated wordlists |
| **fuzzdb** | Attack pattern wordlist collection |
| **Bo0m's SSRF Wordlist** | SSRF parameter names |
| **OneListForAll** | Combined wordlist |

## Payload Collections

### XSS
- **PayloadBox XSS Payloads** — https://github.com/payloadbox/xss-payload-list
- **PortSwigger XSS Cheat Sheet** — https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

### SQL Injection
- **PayloadBox SQLi Payloads** — https://github.com/payloadbox/sql-injection-payload-list
- **swisskyrepo PayloadAllTheThings** — https://github.com/swisskyrepo/PayloadsAllTheThings

### SSRF
- **swisskyrepo SSRF** — https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery
- **Bo0om SSRF Test** — https://github.com/Bo0oM/SSRF-Testing

### SSTI
- **PayloadAllTheThings SSTI** — https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection

### Command Injection
- **PayloadAllTheThings CMDI** — https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection

## Custom Wordlist Generation

```bash
# Generate from existing content
cat urls.txt | unfurl -u path | sort -u > paths.txt
cat urls.txt | unfurl -u keys | sort -u > params.txt

# Generate from JS files
cat all.js | grep -oP '"([a-zA-Z_][a-zA-Z0-9_]*)"' | sort -u > js_keys.txt

# Generate from Wayback
gau --subs target.com | unfurl -u path | sort -u > wayback_paths.txt
```

---

> **Next**: [Books & Channels](04-books-and-channels.md)
