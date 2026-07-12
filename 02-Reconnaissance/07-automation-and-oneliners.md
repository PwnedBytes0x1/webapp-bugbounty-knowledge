# Automation & One-Liners: Production-Ready Pipelines

## Full Automation Script

```bash
#!/bin/bash
DOMAIN=$1
OUTPUT="$HOME/results/$(date +%Y-%m-%d)/$DOMAIN"
mkdir -p $OUTPUT

echo "[*] Phase 1: Passive subdomain enumeration"
subfinder -d $DOMAIN -all -silent | anew $OUTPUT/subs_passive.txt
cat $OUTPUT/subs_passive.txt | httpx -silent -title -tech-detect -status-code > $OUTPUT/live_passive.txt 2>/dev/null

echo "[*] Phase 2: Active subdomain enumeration"
puredns bruteforce /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt $DOMAIN -r resolvers.txt -w $OUTPUT/subs_active_1.txt 2>/dev/null
puredns bruteforce /usr/share/seclists/Discovery/DNS/raft-large-words.txt $DOMAIN -r resolvers.txt -w $OUTPUT/subs_active_2.txt 2>/dev/null

echo "[*] Phase 3: Permutation attack"
cat $OUTPUT/subs_passive.txt $OUTPUT/subs_active_*.txt 2>/dev/null | sort -u | alterx | puredns resolve -r resolvers.txt -w $OUTPUT/subs_permutations.txt 2>/dev/null

echo "[*] Phase 4: Merge and resolve"
cat $OUTPUT/subs_*.txt 2>/dev/null | grep -E '\.'$DOMAIN'$' | sort -u > $OUTPUT/all_subs.txt
dnsx -l $OUTPUT/all_subs.txt -a -cname -resp -silent -o $OUTPUT/resolved.txt 2>/dev/null

echo "[*] Phase 5: HTTP probing"
httpx -l $OUTPUT/resolved.txt -title -tech-detect -status-code -web-server -ip -cname -cdn -content-length -follow-redirects -silent -json -o $OUTPUT/live_hosts.json 2>/dev/null

echo "[*] Phase 6: Port scanning"
naabu -l $OUTPUT/all_subs.txt -top-ports 100 -silent -o $OUTPUT/ports.txt 2>/dev/null

echo "[*] Phase 7: URL collection"
cat $OUTPUT/live_hosts.json 2>/dev/null | jq -r '.url' | gau --threads 5 > $OUTPUT/historical_urls.txt 2>/dev/null

echo "[*] Phase 8: JS analysis"
cat $OUTPUT/historical_urls.txt | grep -E '\.js($|\?)' | sort -u > $OUTPUT/js_files.txt
echo "[*] Done! Output in $OUTPUT"
```

## One-Liners for Quick Testing

### Subdomain Enumeration
```bash
# Combined passive + active in one line
subfinder -d $1 -all -silent | dnsx -a -silent | httpx -silent -title -status-code

# Mass subdomain takeover check
cat subs.txt | dnsx -cname -resp -silent | grep -P '(s3\.amazonaws\.com|cloudfront\.net|herokuapp\.com|github\.io|azurewebsites\.net)'

# Favicon hash extraction + Shodan search
python3 favicon_hash.py -d $1 && curl -s "https://www.shodan.io/search?query=http.favicon.hash%3A$(cat favicon_hash.txt)"

# ASN discovery and reverse DNS
asnmap -d $1 | httpx -silent -resp-only -ptr | sort -u
```

### Content Discovery
```bash
# Directory brute force with common wordlist
ffuf -u https://$1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 100 -fc 403,404 -of json -o ffuf.json

# API endpoint brute force
ffuf -u https://api.$1/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -fc 403,404

# Hidden parameter discovery
cat live_urls.txt | grep '?' | sed 's/?.*//' | sort -u | xargs -I@ sh -c 'ffuf -u "@?FUZZ=1" -w params.txt -fc 400,404'

# Wayback machine URL extraction with session analysis
curl -s "http://web.archive.org/cdx/search/cdx?url=*.$1&output=json&fl=original&collapse=urlkey&limit=100000" | jq -r '.[]' | grep -v '^\[' | sort -u | grep '?' >> wayback_urls.txt
```

### Vulnerability Scanning
```bash
# XSS parameter brute force
cat url_params.txt | while read u; do echo -e "$u'\n$u\"\n$u<script>\n$u<img src=x>"; done | httpx -silent -match-string "<script>" -match-string "<img src=x>"

# SSRF parameter brute force
ffuf -u "https://$1/?FUZZ=http://collaborator.oastify.com" -w params.txt -fw 0

# SQLi parameter brute force
cat url_params.txt | while read u; do curl -s -o /dev/null -w "%{http_code}:%{time_total}" "$u'" && echo " $u"; done

# Open redirect via URL parameter
cat url_params.txt | grep -E '(url=|redirect=|return=|next=|dest=|redirect_uri=|callback=)' | sort -u
```

### JS Analysis
```bash
# Extract all JS endpoints
cat all_js.js | tr ';' '\n' | grep -oP '("|\'|`)(/[a-zA-Z0-9_\/.-]+)("|\'|`)' | sort -u | grep -v 'node_modules'

# Find API keys in JS
curl -s "https://$1/app.js" | grep -oE '("|\')?(api[Kk]ey|apikey|api_key|secret|token|password|auth)("|\')?\s*[:=]\s*("|\')([^"\']+)("|\')'

# Find internal endpoints in JS
curl -s "https://$1/app.js" | grep -oE '"https?://[a-zA-Z0-9._-]+/[a-zA-Z0-9_\/.-]+"' | sort -u
```

### Cloud
```bash
# AWS S3 brute force
while read bucket; do curl -s -o /dev/null -w "%{http_code}" "https://$bucket.s3.amazonaws.com/"; echo " $bucket"; done < bucket_names.txt

# Check for open S3 buckets
s3scanner -bucket-file bucket_names.txt -dump

# GCP bucket check
while read b; do curl -s "https://$b.storage.googleapis.com/" | head -1; done < target_buckets.txt
```

### Monitoring (Cron Job)
```bash
# Run daily, diff against yesterday
0 8 * * * /opt/recon/run.sh target.com /opt/recon/output 2>&1
# Notify on new subdomains
cat output/subs.txt | sort -u > today.txt; diff yesterday.txt today.txt | grep '^>' | mail -s "New subs for target" you@example.com; cp today.txt yesterday.txt
```

## Tool-Specific Optimization

### nuclei
```bash
# Custom template execution with rate limiting
nuclei -t cves/ -l targets.txt -rl 30 -c 10 -timeout 5 -o critical.txt

# Template classification
nuclei -l targets.txt -t http/misconfiguration/ -o misconfigs.txt
nuclei -l targets.txt -t http/exposures/ -o exposures.txt
nuclei -l targets.txt -t http/cves/ -o cves.txt

# Headless mode for JS-heavy targets
nuclei -l targets.txt -t headless/ -headless -o headless_findings.txt
```

### httpx
```bash
# Full metadata capture
httpx -l hosts.txt -title -tech-detect -status-code -content-length -web-server -ip -cname -cdn -response-time -follow-redirects -json -o full_metadata.json

# Screenshot all live hosts
httpx -l hosts.txt -screenshot -screenshot-dir screenshots/
```

### katana
```bash
# Full crawl with JS parsing
katana -u https://$1 -jc -kf all -d 5 -o all_endpoints.txt

# Form discovery for CSRF/parameter testing
katana -u https://$1 -jc -kf form -o forms.txt
```

## Data Flow Pipeline
```
Scope -> Subdomains -> Live Hosts -> URLs -> JS -> Parameters -> Vulns
```

## Output Organization
```
$TARGET/
  recon/
    scopes/
    subs/
    hosts/
    urls/
    js/
    params/
  vulns/
    xss/
    sqli/
    ssrf/
    idor/
    takeover/
  reports/
```
