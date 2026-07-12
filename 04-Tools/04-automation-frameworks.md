# Automation Frameworks: Pipeline Architecture

## Daily Scan Script
```bash
#!/bin/bash
TODAY=$(date +%Y-%m-%d)
SCOPE_FILE="$HOME/scopes/active.txt"
OUTPUT_DIR="$HOME/results/$TODAY"
mkdir -p $OUTPUT_DIR

while read target; do
  echo "[*] Scanning: $target"
  
  # Phase 1: Subdomain discovery
  subfinder -d "$target" -all -silent | anew "$OUTPUT_DIR/$target/subs.txt"
  
  # Phase 2: Resolution + HTTP probing
  cat "$OUTPUT_DIR/$target/subs.txt" | \
    dnsx -a -cname -resp -silent | \
    httpx -title -status-code -tech-detect -silent > "$OUTPUT_DIR/$target/metadata.json"
  
  # Phase 3: Vulnerability scanning
  cat "$OUTPUT_DIR/$target/metadata.json" | jq -r '.url' | \
    nuclei -t $HOME/nuclei-templates/ -severity critical,high -o "$OUTPUT_DIR/$target/nuclei.txt"
  
  # Phase 4: Alert on critical
  if [ -s "$OUTPUT_DIR/$target/nuclei.txt" ]; then
    curl -X POST -H "Content-Type: application/json" \
      -d '{"text":"Critical on '$target': $(cat '$OUTPUT_DIR/$target/nuclei.txt')"}' \
      $SLACK_WEBHOOK_URL
  fi
done < "$SCOPE_FILE"
```

## Notification-Driven Hunting
```bash
# Monitor for new subdomains daily
if [ -f "$OUTPUT_DIR/$target/yesterday.txt" ]; then
  diff "$OUTPUT_DIR/$target/yesterday.txt" "$OUTPUT_DIR/$target/subs_sorted.txt" | \
    grep ">" | mail -s "New subdomains: $target" you@example.com
fi
cp "$OUTPUT_DIR/$target/subs_sorted.txt" "$OUTPUT_DIR/$target/yesterday.txt"
```

## Custom Burp Extension Pattern
```python
from burp import IBurpExtender, IScannerCheck

class BurpExtender(IBurpExtender, IScannerCheck):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.setExtensionName("Custom Scanner")
        callbacks.registerScannerCheck(self)
```
