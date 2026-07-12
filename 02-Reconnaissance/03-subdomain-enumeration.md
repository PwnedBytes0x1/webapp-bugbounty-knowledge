# Subdomain Enumeration: 2026 Complete Reference

## Passive Tools (30+ sources)

| Tool | Sources | Command |
|------|---------|---------|
| subfinder | 30+ passive, API-key aware | subfinder -d target.com -all -silent |
| amass (passive) | Deepest coverage, graph DB | amass enum -passive -d target.com |
| assetfinder | Minimal, fast, pipelining | assetfinder --subs-only target.com |
| chaos | PD curated dataset | chaos -d target.com -silent |
| crt.sh | CT logs | curl -s 'https://crt.sh/?q=%25.target.com&output=json' |
| bbot | 80+ modules, swiss-army | bbot -t target.com -f subdomain-enum |

### subfinder API Configuration
~/.config/subfinder/provider-config.yaml:
Chaos, SecurityTrails, Censys, Shodan, VirusTotal, GitHub, BinaryEdge, WhoisXML

### 47+ Passive Sources (SubHunterX)
- crt.sh, Shodan, VirusTotal, AlienVault, Censys, SecurityTrails, BinaryEdge
- HackerTarget, DNSDumpster, BufferOver, Robtex, CIRCL PassiveDNS
- RapidDNS, Anubis DB, Columbus Project, Riddler.io, DNSrepo

## Active Enumeration

### DNS Brute Force
puredns bruteforce wordlist.txt target.com -r resolvers.txt
shuffledns -d target.com -w wordlist.txt -r resolvers.txt

### DNS NOERROR Detection (ENT Detection)
Standard brute-force misses Empty Non-Terminals (ENTs) - nodes without A/AAAA/CNAME but with subdomains below them.
dnsx -d target.com -w wordlist.txt -rcode noerror  # Catches ENTs
ENTS yield NOERROR with empty answer section. These are directories in the DNS tree.
RFC 8020: ENTs should return NOERROR/NODATA.
Cloudflare "Black Lies" in DNSSEC convert NXDOMAIN->NOERROR (can't distinguish ENTs there).

### Recursive Enumeration
After finding subs, enumerate *.sub.target.com recursively.
subfinder -d sub.target.com -all -silent
puredns bruteforce wordlist.txt sub.target.com

### DNS Zone Transfer
dig AXFR target.com @ns1.target.com
dnsrecon -d target.com -t axfr
IXFR (incremental) sometimes open when AXFR is blocked

## Permutation Generation

### Tools
- dnsgen: Pattern-based mutations from known subdomains
- altdns: Deep permutations with custom wordlist
- gotator: Deep pattern permutations (slower, more thorough)
- ripgen: Fast permutations (faster, less thorough)
- subwiz: AI-powered permutations (reconFTW)
- regulator: Regex-based permutations

### Commands
cat subs.txt | dnsgen - | puredns resolve -r resolvers.txt
altdns -i subs.txt -o altdns.txt -w words.txt -r -s results.txt
gotator -sub subs.txt -perm words.txt -depth 1 -numbers 5 | puredns resolve

### High-Value Permutations
dev-* , *-dev , *-old , *-legacy , *-v1 , *-v2 , *-staging , *-internal , *-api, api-* , *-backup , *-test

## Wildcard Detection
dnsx detects wildcards automatically. Critical before brute-force.
puredns has built-in wildcard filtering.
Amass tracks wildcards in graph database.

## Resolution & Validation
puredns resolve subs.txt -r resolvers.txt -o resolved.txt
dnsx -l subs.txt -resp-only -o resolved.txt
httpx -l resolved.txt -title -status-code -tech-detect -o live.txt

## Resolution + Probe Pipeline
subfinder -d target.com -all | puredns resolve -r resolvers.txt | httpx -silent -title -status-code -tech-detect -o live.txt

## Anti-Detection
- Rate limit DNS queries (puredns --rate-limit)
- Use DoH (DNS-over-HTTPS) resolvers
- Rotate resolvers list
- Add delays between queries
- Use multiple VPS sources