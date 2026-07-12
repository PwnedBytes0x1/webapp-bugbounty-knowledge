# Automation Frameworks: 2026 Reference

## Recon Frameworks

### ProjectDiscovery Unified CLI (pdtm)
Single binary managing all PD tools (nuclei, httpx, subfinder, naabu, katana, uncover, interactsh, chaos, asnmap, dnsx, mapcidr, ppmap). Updates, installs, and runs all tools from one interface.

### reconFTW
Bash-based automation covering:
- Subdomain enumeration (30+ sources)
- DNS resolution with wildcard filtering
- Port scanning (top-1000)
- URL discovery (Wayback, Gau, Katana)
- Vulnerability scanning (Nuclei)
- JavaScript analysis
- Cloud asset discovery

### Other Frameworks
| Framework | Language | Focus |
|-----------|----------|-------|
| bbot | Python | 80+ modules, subdomain -> vuln |
| rengine | Django | Web UI, team workflows |
| axiom | Bash | VPS-based distributed scanning |
| lazyrecon | Bash | Lightweight recon script |
| Sn1per | Bash | Full pentest automation |

## CI/CD Integration

### GitHub Actions
```yaml
- name: Continuous Recon
  run: |
    subfinder -d target.com -silent |       httpx -silent -title -status-code |       nuclei -silent -t ~/nuclei-templates/
```

### Slack/Discord Notifications
Pipe findings from nuclei/httpx into webhook notifiers for real-time alerts.

## Custom Pipeline Advice

1. File-based: Store each stage output for reuse (subs.txt → resolved.txt → live.txt → vulns.txt)
2. Avoid re-running passive recon on every scan — cache results
3. Use parallel execution with GNU parallel or PD's built-in concurrency
4. Tag findings by source for triage priority