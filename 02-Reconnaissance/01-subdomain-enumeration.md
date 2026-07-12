# Subdomain Enumeration: Beyond the Basics

## The Funnel Approach

Subdomain enumeration has three phases, each narrowing the set while increasing value:

### Phase 1: Passive Collection
Run ALL passive sources — not just subfinder. Each source has unique coverage:

```bash
# Passive sources
subfinder -d target.com -all -silent
assetfinder --subs-only target.com
amass enum -passive -d target.com
github-subdomains.py -d target.com
# Certificate Transparency
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u
curl -s "https://crt.sh/?q=%25.target.com&excluded=expired&output=json" | jq -r '.[].name_value' | sort -u
# DNS datasets
curl -s "https://api.securitytrails.com/v1/domain/target.com/subdomains" -H "APIKEY: $KEY"
# CommonCrawl
curl -s "http://index.commoncrawl.org/CC-MAIN-YYYY-WW-index?url=*.target.com&output=json"
```

### Phase 2: Active Probing
DNS brute-force is noisy but catches what passive misses:

```bash
# Use curated wordlists
puredns bruteforce ~/wordlists/commonspeak2-subdomains.txt target.com
# Or massdns for speed
massdns -r ~/tools/massdns/lists/resolvers.txt -t A -o S -w massdns.out commonspeak2-subdomains.txt
# DNS brute with mutations (alterations of known subdomains)
dnsgen <known-subs.txt | massdns -r resolvers.txt -t A -o J 2>/dev/null | jq -r '. | select(.resp_type=="A") | .name'
```

### Phase 3: Permutation & Alteration
Generate permutations of known subdomains:

```bash
# Generate permutations of discovered subdomains
dnsgen discovered_subs.txt | massdns -r resolvers.txt -t A -o J | jq -r '.name' | anew live_subs.txt
# Check for zone transfer (rare but immediate win)
dig axfr @ns1.target.com target.com
```

## CT Log Deep-Dive

Certificate Transparency logs reveal subdomains before they are publicly resolvable. But crt.sh only scratches the surface:

```python
# Query certspotter API for all issuances
curl "https://api.certspotter.com/v1/issuances?domain=target.com&include_subdomains=true&expand=dns_names"
```

Check for:
- **Staging/Dev domains** with TLS certs issued to production CA
- **Expired certs** that reveal deprecated subdomains still resolving
- **Wildcard certs** that reveal the naming convention pattern

## Wildcard Filtering Bypass

Some domains return IP for any subdomain (wildcard DNS). Filter these out:

```bash
puredns resolve subs.txt -r resolvers.txt --wildcard-batch 100000
```

Manual approach: resolve a random subdomain (e.g., `asdfjasldfkj.target.com`). If it resolves, wildcard DNS is active. Filter by comparing resolved IPs against the wildcard baseline.

## Resolver Selection

Public resolvers (8.8.8.8, 1.1.1.1) rate-limit aggressively. Use:
- `dnsvalidator` to generate a fresh resolver list
- Rotate pools per scan phase
- Target ~1000 resolvers for massdns to avoid rate limiting

## Service Discovery on Discovered Subdomains

```bash
# Check HTTP/HTTPS with metadata
httpx -l subs.txt -title -status-code -tech-detect -follow-redirects -silent
# Port scan top services on discovered hosts
naabu -l live_hosts.txt -top-ports 1000 -silent | httpx -silent
```

Key signals:
- **Staging/dev** subdomains often have weaker security + test data
- **403/401** endpoints that differ from production (auth gap)
- **Technology shifts** (different framework, older versions, debug mode enabled)
