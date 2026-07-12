# Automation Frameworks

## ProjectDiscovery Ecosystem
```bash
subfinder -d target.com | httpx -o live.txt
katana -list live.txt -o urls.txt
nuclei -l live.txt -t ~/nuclei-templates/
```

## Orchestration Tools
| Tool | Description |
|------|-------------|
| Reconftw | Complete recon framework with 70+ tools |
| reNgine | Web-based recon with Django |
| recon-ng | Full-featured recon framework |
| sn0int | OSINT framework with plugins |

## CI/CD Integration
```yaml
name: Nightly Recon
on:
  schedule:
    - cron: '0 6 * * *'
jobs:
  recon:
    runs-on: ubuntu-latest
    steps:
      - run: subfinder -d target.com -o subs.txt
      - run: httpx -l subs.txt -o live.txt
      - uses: actions/upload-artifact@v3
        with:
          name: recon-results
          path: ./*.txt
```

## Custom Pipeline
```python
import subprocess, json
domain = "target.com"
subs = subprocess.run(["subfinder", "-d", domain, "-silent"], capture_output=True, text=True)
with open("subs.txt", "w") as f:
    f.write(subs.stdout)
live = subprocess.run(["httpx", "-l", "subs.txt", "-silent", "-json"], capture_output=True, text=True)
for line in live.stdout.strip().split("
"):
    if line:
        data = json.loads(line)
        print(f"Live: {data['url']} - {data.get('title', 'N/A')}")
```

## Notification Integration
- **notify** - Send Slack/Discord/Telegram notifications
- **apprise** - Multi-platform notification library

```bash
subfinder -d target.com -silent | anew subs.txt | notify -provider slack
```

---

> **Next Section**: [Advanced Topics](../05-Advanced-Topics/01-cloud-security.md)
