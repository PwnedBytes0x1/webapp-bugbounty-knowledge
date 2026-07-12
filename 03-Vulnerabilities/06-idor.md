# Insecure Direct Object References (IDOR): 2026 Complete Reference

## Attack Vectors

### Numeric ID Enumeration
- `/api/users/1`, `/api/users/2`, `/api/users/3`
- UUID/GUID enumeration: `/api/invoices/550e8400-e29b-41d4-a716-446655440000`
- Increment by unusual step sizes: `/api/users/1000`, `/api/orders/10000`
- Parallel enumeration (rate limit bypass): send 100 concurrent requests with different IDs

### UUID Predictability
- UUIDv1: timestamp-based, predictable if you know creation time
- UUIDv4: theoretically random — check if using `Math.random()` (not cryptographically secure)
- Sequential UUIDs: some implementations use seeded PRNG
- Auto-increment hashes: MD5/SHA1 of numeric ID — still enumerable

### Mass Assignment
```json
// API accepts extra parameters that modify object relationships
POST /api/order
{"product_id": 5, "quantity": 1, "user_id": 999}  // Attacker sets user_id to another user
```

### Path Traversal IDOR
- `/documents/../../etc/passwd`
- `/api/files/../../../etc/passwd`
- `/download?file=../../config/database.yml`

### Parameter Pollution
- `GET /api/order?id=1337&id=9999` (some frameworks use the second)
- `GET /api/data?uid=123¶m=value¶m=value` (array injection)

## Detection Methodology
1. Authorize as User A → access resource of User B by modifying identifier
2. Try alternate identifier formats: numeric vs UUID vs hash
3. Check different HTTP methods on same endpoint: `DELETE /api/user/1`
4. Look for ID in: URL path, query params, POST body, headers, cookies
5. Try path traversal patterns on file download endpoints
6. Check for ID in GraphQL mutations

## Common Bounty Findings
| Pattern | Payload | Bounty |
|---------|---------|--------|
| User ID enumeration | Change `/users/me` to `/users/123` | $500-$2000 |
| Order details | Change `order_id` in API response | $500-$3000 |
| Document access | Change `file_id` parameter | $1000-$5000 |
| Admin panel IDOR | Remove admin check but keep ID parameter | $2000-$10000+ |

## Defensive Checklist
1. Use indirect object references (GUIDs for external, numeric internally)
2. Add server-side authorization checks for EVERY resource access
3. Use UUIDv4 from crypto-secure RNG (not Math.random)
4. Implement rate limiting on ID-based endpoints
5. Validate user owns resource before returning data
6. Use GraphQL DataLoader with auth context