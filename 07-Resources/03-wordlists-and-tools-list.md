# Wordlists & Tools Reference

## Essential Wordlists

### Content Discovery
```bash
# Directory/file brute-force
/usr/share/wordlists/dirb/common.txt           # ~4,600 entries
/usr/share/seclists/Discovery/Web-Content/      # Full SecLists collection
SecLists/Discovery/Web-Content/raft-large-words.txt  # ~140k entries

# Parameters
SecLists/Discovery/Web-Content/burp-parameter-names.txt
# Custom: all common params + framework-specific (Laravel params, Spring params)
```

### Subdomains
```bash
SecLists/Discovery/DNS/subdomains-top1million-110000.txt
# Custom: curated from real bounty targets (~150k entries)
# Best: combine multiple wordlists + jq filter unique
```

### XSS Payloads
```
PayloadsAllTheThings/XSS Injection/
# Polyglots, event handlers, mXSS, dom clobbering
```

### SQLi Payloads
```
# Time-based, error-based, union, blind, stacked queries
PayloadsAllTheThings/SQL Injection/
```

### Username/Password
- **SecLists/Passwords/CommonCredentials/** - top 100k passwords
- **SecLists/Passwords/darkc0de.txt** - darkc0de list
- **rockyou.txt** - classic (~14GB, 14M passwords)
- **Have I Been Pwned passwords** - 613M passwords (NLP processed)

## Must-Have Repos

| Repo | Purpose |
|------|---------|
| PayloadsAllTheThings | Comprehensive payload collection |
| SecLists | Ultimate wordlist collection |
| HackTricks | Technique encyclopedia |
| swisskyrepo/PayloadsAllTheThings | Payloads by vulnerability |
| OWASP/CheatSheetSeries | OWASP cheat sheets |
| ihebski/DefaultCreds-cheat-sheet | Default credentials |

## Custom Wordlist Generation

### From Target
```bash
# Extract words from JS files
cat all.js | grep -ohP '"[a-zA-Z0-9_-]+"' | sort -u > custom_words.txt
# Extract from source
cat source.html | grep -ohP '"[a-zA-Z0-9_-]+"' | sort -u > html_words.txt
# Combine with common wordlists
cat custom_words.txt common.txt | sort -u > combined.txt
```

### CeWL (Custom Word List)
```bash
cewl -d 2 -m 5 https://target.com -w target_words.txt
```
