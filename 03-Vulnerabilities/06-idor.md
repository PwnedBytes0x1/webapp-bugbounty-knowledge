# Insecure Direct Object References (IDOR)

IDOR (Insecure Direct Object Reference) occurs when an application exposes a reference to an internal implementation object — such as a file, database key, or record — without proper access control checks. This is the most common bug bounty finding.

## How IDOR Works

```http
GET /api/v1/users/1234/profile HTTP/1.1
Authorization: Bearer user_token
```

If user 1234 can access user 5678's profile by simply changing the ID, that's an IDOR.

## Common IDOR Patterns

### Numeric ID
```http
GET /invoice/1001  →  GET /invoice/1002
POST /api/order/delete
{"order_id": 5001}  →  {"order_id": 5002}
```

### UUID/GUID (Predictable)
```http
GET /api/document/a1b2c3d4-e5f6-7890-abcd-ef1234567890
```
If UUIDs are sequential or created at predictable times, they can be brute-forced.

### Hash-based IDs
```http
GET /profile/MTIzNA==    (base64 of "1234")
GET /profile/MTIzNQ==    (base64 of "1235")
```

### Email/Username as ID
```http
GET /api/user?email=admin@target.com
GET /api/user?email=other@target.com
```

## Types of IDOR

| Type | Description |
|------|-------------|
| **Horizontal** | Access another user's data at same privilege level |
| **Vertical** | Access data requiring higher privileges |
| **Object-level** | Access specific objects (documents, orders, messages) |
| **Function-level** | Access restricted functions |

## Detection Methodology

### Automated Discovery
```bash
# Find potential IDOR parameters
cat urls.txt | grep -E "(id=|user=|account=|profile=|order=|invoice=|document=|file=|download=)" | sort -u

# Use ffuf to enumerate IDs
ffuf -u "https://target.com/api/users/FUZZ/profile" -w ids.txt -fw 123

# Use Burp Intruder with pitchfork attack
```

### Manual Testing
1. **Create two accounts** (or use collaboration)
2. **Authenticate as User A**, access resource
3. **Note the resource identifier** in URL or API endpoint
4. **Replace identifier** with User B's resource identifier
5. **Check if User A can access User B's data**

## Advanced IDOR Examples

### Multi-step IDOR
```http
# Step 1: Create order
POST /api/orders
{"product_id": 123, "user_id": 100}

# Step 2: Change user_id to different user
POST /api/orders
{"product_id": 123, "user_id": 200}
```

### UUID Enumeration
```http
# Predictable UUID generation
GET /api/users/register    # Server sends welcome email with UUID
UUID = hash(timestamp + email)

# SSRF to internal IDOR
GET /internal/api/users?id=123
```

### Parameter Pollution
```http
# Both parameters might be evaluated
GET /api/users?id=100&user_id=200

# JSON array injection
POST /api/users/delete
{"user_ids": [100, 200]}
```

## Tools

| Tool | Purpose |
|------|---------|
| **Autorize** (Burp) | Detect ATO/IDOR automatically |
| **AuthMatrix** (Burp) | Role-based access testing |
| **Burp Intruder** | ID enumeration |
| **ffuf** | Fuzz IDs across users |
| **Custom Python** | Sequential ID guessing with auth tokens |

## Prevention

1. **Object-level access controls** — Verify user owns the resource
2. **Use unpredictable identifiers** — UUIDv4 (non-sequential)
3. **Don't expose internal IDs** — Use indirect references
4. **Server-side enforcement** — Never rely on client-side access control
5. **Comprehensive API testing** — Every endpoint needs access control checks

---

> **Next**: [Authentication Bypass](07-authentication-bypass.md)
