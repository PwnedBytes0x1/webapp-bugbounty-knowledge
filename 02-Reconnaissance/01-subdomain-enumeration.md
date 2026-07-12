# Subdomain Enumeration: Complete Field Guide

## Phase 1: Passive Enumeration (Zero Interaction)

### Certificate Transparency (CT) Logs
CT logs are the single most effective passive source. Every TLS certificate issued by a public CA is logged here.

```bash
# crt.sh - main CT log index
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u > crtsh_subs.txt

# CertSpotter (SSLMate)
curl -s "https://certspotter.com/api/v0/certs?domain=example.com" | jq -r '.[].dns_names[]' | sort -u

# Censys search (requires API key)
censys search "services.tls.certificates.leaf_data.subject.common_name: example.com AND tags: \`trusted\`" --pages 5

# Google Transparency Report
curl -sL "https://transparencyreport.google.com/transparencyreport/api/v3/httpsreport/ct/certsearch?domain=example.com&include_subdomains=true"
```

### DNS Aggregator Services
```bash
# subfinder (30+ sources)
subfinder -d example.com -all -silent -recursive -o subfinder.txt
# Configure API keys: ~/.config/subfinder/provider-config.yaml

# amass passive (slower, deeper)
amass enum -passive -d example.com -o amass_passive.txt

# assetfinder (fast, crt.sh + ThreatCrowd + CertSpotter)
assetfinder --subs-only example.com > assetfinder_subs.txt

# findomain (SecurityTrails + VirusTotal + certspotter)
findomain -t example.com -u findomain_subs.txt

# chaos (ProjectDiscovery's curated DNS dataset)
chaos -d example.com -silent -o chaos_subs.txt

# github-subdomains (scrapes GitHub code search)
github-subdomains -d example.com -t $GITHUB_TOKEN > github_subs.txt

# bbot (80+ modules, all-in-one)
bbot -t example.com -f subdomain-enum -o bbot_output
```

### Search Engine Dorking
```text
site:*.example.com -www -mail
site:example.com intitle:"index of"
inurl:example.com filetype:pdf confidential
site:pastebin.com example.com
"example.com" "api_key" OR "password" OR "secret"
```

## Phase 2: Active Enumeration

### DNS Brute Force
```bash
# puredns (wildcard-aware, gold standard)
puredns bruteforce /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt example.com -r resolvers.txt -w puredns_results.txt

# massdns (raw performance)
massdns -r resolvers.txt -t A -o S -w massdns_output.txt wordlist.txt

# shuffledns (projectdiscovery, simpler)
shuffledns -d example.com -list target_subs.txt -r resolvers.txt -o shuffledns_results.txt
```

### Resolver Requirements
```bash
# Get fresh resolvers with dnsvalidator
dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 20 -o resolvers.txt
```

### Wildcard Detection
```bash
# Test for wildcard first
dig +short A random123subdomain.example.com
# If it resolves, wildcard is active -> use puredns, not dnsx
dnsx -l all_subs.txt -a -wd example.com -o resolved.txt
```

## Phase 3: Permutation Attack

```bash
# alterx (fast, projectdiscovery)
cat discovered_subs.txt | alterx | dnsx -a -silent -o permutations_resolved.txt

# dnsgen (comprehensive, slower)
dnsgen discovered_subs.txt -w permutations_list.txt | massdns -r resolvers.txt -t A -o S | grep " A " | cut -d' ' -f1 > new_discovered.txt

# gotator (deep permutation with numbers)
gotator -sub discovered_subs.txt -perm /usr/share/seclists/Discovery/DNS/dns-Jhaddix.txt -depth 1 -numbers 5 -md | puredns resolve -r resolvers.txt -w gotator_resolved.txt
```

### Permutation Patterns
- Prefix/suffix: dev-, -dev, api-, -staging, -v2, -old, -test, -backup
- Number: api2, api3, backend01, web02
- Tech stack: k8s-, kube-, -eks, -ecs, -lambda, -cdn
- Environment: prod-, stag-, stage-, uat-, pilot-, canary-
- Service: grafana., jenkins., gitlab., nexus., jira., confluence., sonar.
- Region: us-east-1., eu-west-1., ap-southeast-1.

## Phase 4: ASN & Infrastructure Expansion

```bash
# Find ASN
asnmap -d example.com
# or use whois
whois -h whois.radb.net -- '-i origin AS13335'

# Enumerate all IPs in ASN
curl -s "https://api.bgpview.io/asn/13335/prefixes" | jq -r '.data.ipv4_prefixes[].prefix'

# Reverse DNS on all IPs
cat cidr_ranges.txt | dnsx -silent -resp-only -ptr > ptr_records.txt

# Shodan search by ASN
asn:AS13335 org:"Example" country:US

# Censys search by ASN
services.service_name: HTTP AND autonomous_system.asn: 13335
```

## Phase 5: Virtual Host Discovery

```bash
# HTTP VHOST fuzzing
ffuf -u https://TARGET_IP -w subs.txt -H "Host: FUZZ.example.com" -fc 200,301,302,400 -ac

# DNS-agnostic VHOST discovery
gobuster vhost -u https://example.com -w vhost_wordlist.txt
```

## Phase 6: Merging & Verification

```bash
# Merge all sources
cat *_subs.txt *_results.txt *_resolved.txt 2>/dev/null | grep -E '\.example\.com$' | sort -u > all_discovered.txt
wc -l all_discovered.txt

# DNS resolution
cat all_discovered.txt | dnsx -a -cname -resp -silent -o resolved_hosts.txt

# HTTP probing
cat resolved_hosts.txt | httpx -title -tech-detect -status-code -web-server -ip -cname -cdn -silent -o live_hosts.json
```

## Tool Configuration

### subfinder provider-config.yaml
```yaml
securitytrails: [API_KEY]
virustotal: [API_KEY]
shodan: [API_KEY]
censys: [API_KEY, SECRET]
chaos: [API_KEY]
binaryedge: [API_KEY]
github: [GITHUB_TOKEN]
passivetotal: [USERNAME, API_KEY]
whoisxmlapi: [API_KEY]
```
