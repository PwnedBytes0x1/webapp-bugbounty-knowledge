# Automation Frameworks & Pipelines

## Custom Recon Pipeline (Bash)
```bash
#!/bin/bash
DOMAIN=$1
mkdir -p $DOMAIN/{recon,scanning,exploitation,reporting}

# Subdomain enumeration
subfinder -d $DOMAIN -o $DOMAIN/recon/subs.txt
amass enum -d $DOMAIN -o $DOMAIN/recon/amass.txt
sort -u $DOMAIN/recon/*.txt > $DOMAIN/recon/all-subs.txt

# Probe alive hosts
cat $DOMAIN/recon/all-subs.txt | httpx -o $DOMAIN/recon/live-subs.txt

# Screenshot
cat $DOMAIN/recon/live-subs.txt | aquatone -out $DOMAIN/recon/screenshots

# Nuclei scan
nuclei -l $DOMAIN/recon/live-subs.txt -o $DOMAIN/scanning/nuclei.txt

# Parameter discovery
cat $DOMAIN/recon/live-subs.txt | while read url; do
  waybackurls $url | uro >> $DOMAIN/recon/params.txt
done

# XSS scanning
cat $DOMAIN/recon/params.txt | dalfox pipe -o $DOMAIN/exploitation/xss.txt

# Generate report
python3 generate_report.py $DOMAIN
```

## Burp Automation (Python with Montoya API)
```python
# Burp extension for automated scanning
from burp import IBurpExtender, IScannerCheck

class BurpExtender(IBurpExtender):
    def registerExtenderCallbacks(self, callbacks):
        self.callbacks = callbacks
        callbacks.setExtensionName("Auto-Intruder")
        callbacks.registerScannerCheck(self)
```

## Automation Tools

### httpx
Fast HTTP probe for alive hosts
`cat subs.txt | httpx -title -status-code -tech-detect`

### gau/waybackurls
Fetch known URLs from Wayback Machine
`echo target.com | gau | uro > urls.txt`

### uro
URL deduplication tool
`cat urls.txt | uro`

### unfurl
Extract URL components for analysis
`cat urls.txt | unfurl -u format %p%?%q`

## CI/CD Integration

### GitHub Actions Recon
```yaml
name: Automated Recon
on: [push, workflow_dispatch]
jobs:
  recon:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Subfinder
        run: subfinder -d ${{ secrets.DOMAIN }}
      - name: Nuclei Scan
        run: nuclei -u https://${{ secrets.DOMAIN }}
```