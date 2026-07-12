# Server-Side Request Forgery (SSRF)

## What is SSRF?

SSRF occurs when an application fetches a remote resource based on user-supplied URL without proper validation.

## Critical Cloud Metadata Targets

| Provider | URL | Impact |
|----------|-----|--------|
| AWS | http://169.254.169.254/latest/meta-data/ | IAM credentials |
| GCP | http://metadata.google.internal | Service account keys |
| Azure | http://169.254.169.254/metadata/instance | Managed identity |
| Alibaba | http://100.100.100.200 | Instance metadata |
| Docker | http://localhost:2375 | Container escape |
| K8s | https://kubernetes.default.svc | K8s API access |

## Testing Methodology

1. Blind SSRF detection with Interactsh/Burp Collaborator
2. Protocol smuggling (gopher, file, dict)
3. Cloud metadata extraction
4. Internal service discovery

## SSRF to RCE Chains

- SSRF -> Cloud Metadata -> IAM Credentials -> Cloud Console Access
- SSRF -> Internal Redis -> Write SSH Key -> Server Access
- SSRF -> K8s API -> POD Creation -> Container Access

## Tools
- Interactsh (OOB detection)
- Burp Collaborator
- SSRFmap
- Gopherus

## Prevention
- Allowlist of permitted URLs/protocols
- Block private IP ranges
- Disable unnecessary URL schemes (file, gopher, dict)
