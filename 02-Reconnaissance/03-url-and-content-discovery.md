# URL & Content Discovery

## Approach

### 1. Crawling & Spidering
- Tools: katana, gospider, dalfox, Burp Spider

### 2. Wayback & Historical URLs
- gau --subs target.com | sort -u
- waybackurls, meg

### 3. Directory & File Brute-Forcing
- Tools: ffuf, gobuster, dirsearch, feroxbuster
- Wordlists: directory-list-2.3-medium, raft-small, big.txt

### 4. Parameter Fuzzing
- Tools: ffuf, arjun, ParamSpider, x8

### 5. JavaScript Source Maps
- .map files can reveal source code with endpoints and API keys

## ffuf Examples
```bash
# Directory discovery
ffuf -u https://target.com/FUZZ -w wordlist.txt -mc 200,403,301

# Virtual host discovery
ffuf -u https://target.com -H "Host: FUZZ.target.com" -w vhosts.txt -fs 1234

# Parameter fuzzing
ffuf -u https://target.com/page?FUZZ=1 -w params.txt -fw 123
```

## Response Analysis
| Status | Meaning |
|--------|---------|
| 200 | OK - resource exists |
| 301/302 | Redirect - valid path |
| 403 | Forbidden - exists, worth investigation |
| 401 | Unauthorized - requires auth |
| 405 | Method not allowed - try different method |
| 500 | Server error - possible injection |

---

> **Next**: [JavaScript Analysis](04-javascript-analysis.md)
