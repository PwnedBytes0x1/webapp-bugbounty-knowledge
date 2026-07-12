# Security Headers Reference

## Essential Headers

### Strict-Transport-Security (HSTS)
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
max-age=0  # Disable HSTS (for testing)
```

### Content-Security-Policy (CSP)
```
# Restrictive
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'

# Unsafe (vulnerable to XSS)
Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval'

# Nonce-based
Content-Security-Policy: default-src 'self'; script-src 'nonce-abc123'

# Common bypass targets
script-src: 'unsafe-inline' -> XSS possible
script-src: cdn.example.com -> XSS if CDN has user content
script-src: *.example.com -> XSS from any subdomain with upload
```

### X-Frame-Options
```
X-Frame-Options: DENY              # Most secure
X-Frame-Options: SAMEORIGIN        # Allowed on same origin
Missing -> clickjacking vulnerable
```

### X-Content-Type-Options
```
X-Content-Type-Options: nosniff
# Prevents MIME sniffing
```

### Referrer-Policy
```
Referrer-Policy: no-referrer
Referrer-Policy: same-origin
Referrer-Policy: strict-origin-when-cross-origin  # Default
```

### Permissions-Policy
```
Permissions-Policy: camera=(), microphone=(), geolocation=()
# Controls browser feature access
```

## Headers Check Tools
- securityheaders.com
- Mozilla Observatory
- Burp Active Scan++
- nuclei -t ~/nuclei-templates/misconfiguration/security-headers.yaml