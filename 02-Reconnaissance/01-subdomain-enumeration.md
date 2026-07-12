# Subdomain Enumeration

Subdomain enumeration identifies subdomains for a target domain, expanding the attack surface.

## Passive Enumeration
- **Certificate Transparency** - crt.sh, Certspotter, Censys
- **Search Engines** - Google dork: site:*.target.com -www
- **DNS Databases** - SecurityTrails, Shodan, VirusTotal
- **Wayback Machine** - gau, waybackurls, katana
- **ASN Enumeration** - amass intel, metabigor

## Active Enumeration
- **DNS Brute-Force** - puredns, massdns, shuffledns
- **Permutation Scanning** - dmut, gotator, alterx
- **Zone Transfer** - dig axfr @ns1.target.com target.com

## Workflow
```
subfinder -d target.com | tee subs_passive.txt
puredns bruteforce wordlist.txt target.com | tee subs_brute.txt
cat subs_passive.txt subs_brute.txt | sort -u | httpx -o live_subs.txt
```

## Tools Comparison
| Tool | Type | Best For |
|------|------|----------|
| subfinder | Passive | Quick passive collection |
| amass | Both | Thoroughness (ASN, APIs) |
| puredns | Active | Brute-force with wildcard filtering |
| massdns | Active | Large wordlists at high speed |

## Wordlists
- SecLists/Discovery/DNS/ - multiple sizes
- commonspeak2-subdomains
- jhaddix/all.txt (2M+ entries)

---

> **Next**: [DNS & Network Recon](02-dns-and-network-recon.md)
