# Automation & One-liners

## Essential One-liners

```bash
# Passive subdomain collection
subfinder -d target.com -silent | anew subs.txt

# Live host check
cat subs.txt | httpx -silent -o live.txt

# URL collection
cat live.txt | gau --subs | anew urls.txt
katana -list live.txt -silent -o katana_urls.txt

# JavaScript extraction
cat all_urls.txt | grep -E "\\.js(\\?|$)" | sort -u > js.txt

# Potential SSRF parameters
cat all_urls.txt | grep -E "(url|path|dest|redirect|uri|file|load|read|domain)" | sort -u > ssrf_params.txt

# Potential IDOR endpoints  
cat all_urls.txt | grep -E "(id=|user=|account=|profile=|order=|invoice=)" | sort -u > idor_endpoints.txt
```

## Full Recon Pipeline
```bash
#!/bin/bash
DOMAIN=$1
OUTPUT="recon-$DOMAIN-$(date +%Y%m%d)"
mkdir -p $OUTPUT

subfinder -d $DOMAIN -silent > $OUTPUT/subfinder.txt
httpx -l $OUTPUT/subfinder.txt -silent -o $OUTPUT/live.txt
gau --subs $DOMAIN -o $OUTPUT/gau_urls.txt
katana -list $OUTPUT/live.txt -silent -o $OUTPUT/katana_urls.txt
gowitness file -f $OUTPUT/live.txt -P $OUTPUT/screenshots/
naabu -list $OUTPUT/live.txt -top-ports 1000 -o $OUTPUT/ports.txt
echo "Done! Check $OUTPUT/"
```

## Monitoring for Changes
```bash
diff previous_subs.txt current_subs.txt | grep "^>" > new_subs.txt
```

---

> **Next Section**: [Vulnerabilities](../03-Vulnerabilities/01-owasp-top-10-2025.md)
