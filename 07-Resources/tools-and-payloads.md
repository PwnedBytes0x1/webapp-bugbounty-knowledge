# Tools and Payloads Reference

## Essential Wordlists

| Wordlist | Source | Use |
|----------|--------|-----|
| SecLists | Daniel Miessler | All-purpose |
| Assetnote | assetnote.io | API endpoints, subdomains |
| FuzzDB | OWASP | Fuzzing patterns |
| PayloadsAllTheThings | GitHub | Vuln-specific payloads |

## Installation

```bash
git clone https://github.com/danielmiessler/SecLists /opt/seclists
git clone https://github.com/swisskyrepo/PayloadsAllTheThings
wget -r https://wordlists-cdn.assetnote.io/data/
```

## Custom Wordlist Generation

```bash
gau target.com | unfurl -u paths | sort -u > custom_paths.txt
katana -u https://target.com -silent | unfurl -u paths | sort -u
```

## Key Learning Resources

### Books
- Web Application Hacker's Handbook
- Bug Bounty Bootcamp (Vickie Li)
- Real-World Bug Hunting (Peter Yaworski)
- Web Hacking 101 (Peter Yaworski)

### YouTube Channels
- The XSS Rat
- InsiderPhD
- NahamSec
- STOK
- John Hammond
- IppSec

### Labs
- PortSwigger Web Security Academy (free)
- TryHackMe
- HackTheBox
- PentesterLab
