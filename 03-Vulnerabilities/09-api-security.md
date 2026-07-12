# API Security Testing: REST & GraphQL

## API Discovery

```bash
# Find API endpoints from JS
grep -roP '["''](https?://[^"'']+/api/[^"'']+)["'']' ~/js-files/
# Common patterns
ffuf -u https://target.com/api/vFUZZ -w numbers.txt -fc 404
ffuf -u https://api.target.com/vFUZZ -w numbers.txt -fc 404
# GraphQL endpoint discovery
ffuf -u https://target.com/FUZZ -w graphql-points.txt
```

## REST API Testing Methodology

### HTTP Method Testing
```http
# Test every endpoint with all methods
OPTIONS /api/resource
GET /api/resource
POST /api/resource
PUT /api/resource
DELETE /api/resource
PATCH /api/resource
# Method override headers
X-HTTP-Method-Override: DELETE
X-HTTP-Method: PUT
X-Method-Override: PATCH
```

### Content-Type Switching
```http
# Switch between JSON and XML parsers
Content-Type: application/json
Content-Type: application/xml
Content-Type: text/plain
Content-Type: application/x-www-form-urlencoded
# XML parser may have XXE, JSON parser may be safe
```

### Mass Assignment
```http
POST /api/user/update
{"name": "test", "role": "admin", "isVerified": true, "balance": 999999}
# Test extra fields that the endpoint should not accept
```

### Rate Limiting Bypass
```bash
# IP rotation headers
X-Forwarded-For: $i
X-Real-IP: $i
X-Originating-IP: $i
# GraphQL batching bypass
# Authentication endpoint rate limiting via alias abuse
mutation { a: login(...), b: login(...), ... z: login(...) }
```

## API Versioning Attacks

```bash
# Test older API versions that may lack auth
ffuf -u https://api.target.com/vFUZZ/ -w numbers.txt
# /v1/ vs /v2/ — v1 often has fewer security controls
# /v3/ might be in beta with debug mode
```

## Unauthenticated Endpoint Discovery

```bash
# Test each endpoint found in JS with all auth removed
# Check for:
# - Health/status endpoints
# - Swagger/OpenAPI docs
# - GraphQL introspection
# - Metrics (Prometheus)
# - Debug endpoints
```
