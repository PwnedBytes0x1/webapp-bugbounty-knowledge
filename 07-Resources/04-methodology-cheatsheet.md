# Methodology Cheatsheet

## Core Pipeline
```
Domain → Passive Subs → DNS Resolve → Port Scan → HTTP Probe → Vuln Scan
```

## Quick Commands

### Phase 1: Passive
```bash
subfinder -d $1 -all -silent | tee subs.txt
cat subs.txt | waybackurls | uro | tee urls.txt
```

### Phase 2: Active
```bash
cat subs.txt | puredns resolve -r resolvers.txt | tee resolved.txt
naabu -l resolved.txt -top-ports 1000 -silent -json | tee ports.json
cat ports.json | jq -r '.ip + ":" + (.port|tostring)' | httpx -silent -title -status-code -tech-detect -o live.txt
```

### Phase 3: Content
```bash
cat live.txt | ffuf -u FUZZ -w raft-large-directories.txt -ac -o ffuf.json
cat live.txt | katana -js-crawl -d 3 -silent | tee endpoints.txt
```

### Phase 4: Vuln Scan
```bash
nuclei -l live.txt -t ~/nuclei-templates/ -severity critical,high -o critical.txt
```

## By Vulnerability Type

| Vuln | Quick Check |
|------|-------------|
| XSS | `<script>alert(1)</script>` or `"><img src=x onerror=alert(1)>` |
| SQLi | `' OR 1=1--` or `" OR 1=1--` |
| SSTI | `{{7*7}}` or `${7*7}` |
| SSRF | `http://169.254.169.254/latest/meta-data/` |
| IDOR | Change numeric ID or UUID in URL/body |
| CORS | Add `Origin: https://evil.com` header |