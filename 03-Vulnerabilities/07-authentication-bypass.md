# Authentication & Authorization Bypass: 2026 Complete Reference

## OAuth2 Bypass Techniques

### CSRF on OAuth Flow
- OAuth `state` parameter not validated
- Attacker initiates OAuth flow with their account, intercepts `code`, and sends link to victim
- Victim authorizes attacker's session — attacker now has victim privileges

### Redirect URI Manipulation
- `redirect_uri=https://app.com/callback` → `redirect_uri=https://attacker.com`
- Open redirect in `redirect_uri` validator
- Partial URL override: `redirect_uri=https://app.com/callback@attacker.com`
- Path traversal in `redirect_uri` validation

### Token Theft
- `code` returned in URL fragment (`#access_token=xxx`) — leaked via Referer header
- `code` in query string logged by analytics
- `code` stored in browser history

## SAML Attacks

### XML Signature Wrapping
- Insert multiple assertions, some with valid signature, some with forged content
- Weak XML parser picks wrong assertion
- `pacsol` (Python SAML tool): automates signature wrapping attacks

### XML Signature Exclusion
- `Response` element signed, `Assertion` not signed — modify assertion
- Strip signature entirely if validation is missing

## JWT Attacks

### Algorithm Confusion
- `alg: none` — accept unsigned tokens (pre-2020 libraries still vulnerable)
- `alg: HS256` with RSA public key as HMAC secret
- `alg: RS256` → `alg: HS256` (if server accepts symmetric key)
- `jku` header: point to attacker-controlled JWK Set
- `kid` injection: `"kid": "../../../../dev/null"` or SQLi in KID lookup

### Weak Key Attacks
- Crack JWT with `hashcat -m 16500 jwt.txt wordlist.txt`
- `jwt_tool`: `python jwt_tool.py -d jwt.txt -p rockyou.txt`
- Empty key: `alg: HS256`, key: `""`

### Claims Padding
- Add `iat`, `nbf`, or `exp` far in past/future
- Remove `exp` entirely
- Change `sub` to `admin` or other high-privilege user

## MFA Bypass

### 2FA Fatigue
- Send repeated push notifications until user accepts one
- Social engineer via help desk: "I lost my phone"

### Backup Code Abuse
- Backup codes generated but not invalidated after use
- Infinite backup code generation
- Backup code list shared across sessions

### SMS/Email Interception
- SS7 attacks (SMS interception)
- Email account takeover → password reset links
- SIM swap attack

### Biometric Bypass
- Race condition between biometric API calls
- Fallback to PIN bypasses biometric requirement
- Biometric sensor replay

## Password Reset Attacks

### Token Predictability
- `reset_token=md5(timestamp)` — calculate timestamp
- `reset_token=base64(username)` — decode
- `reset_token=sequential_integer` — enumerate other user tokens

### Host Header Injection in Reset Link
- Email contains: `https://[Host]/reset?token=xxx`
- If Host header is attacker-controlled, reset link points to attacker server
- Chain with X-Forwarded-Host injection

### Timing-Based Enumeration
- Registered email returns faster response
- Different error message for registered vs unregistered
- Rate limiting applied only to correct emails