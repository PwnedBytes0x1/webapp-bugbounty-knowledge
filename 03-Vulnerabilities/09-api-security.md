# API Security: 2026 Complete Reference

## GraphQL Attack Surface

### Introspection
```graphql
query { __schema { types { name fields { name } } } }
```
- Default GraphiQL tool available at `/graphql`, `/graphiql`, `/playground`
- Introspection disabled but field-based guessing works
- Batching attacks via aliases: `query { u1: user(id:1), u2: user(id:2), ... }`

### GraphQL Batching
```graphql
query {
  a: user(id:1) { email, role }
  b: user(id:2) { email, role }
  c: user(id:3) { email, role }
  // ... up to hundreds of queries in single request
}
```
Each query produces separate response. Single request = many users enumerated.

### Mass Assignment (GraphQL)
```graphql
mutation {
  createUser(name: "test", role: "admin", isAdmin: true)
  // Unfiltered mutation accepts fields that should be read-only
}
```

## REST API Vulnerabilities

### Endpoint Discovery
- Shadow endpoints: `/api/v2/user`, `/api/internal`, `/api/private`
- Version traversal: `/api/v1/user/1`, `/api/v2/user/1` (v1 lacks auth)
- Verb tunneling: `POST /api/users?X-HTTP-Method-Override=DELETE`

### JSON Injection
```json
{"__proto__": {"isAdmin": true}}
{"constructor": {"prototype": {"isAdmin": true}}}
{"username": "admin", "role": "admin"}  // Included in request but not intended field
```

### Status Code Fuzzing
- 200 = authorized + exists
- 403 = authorized + exists but forbidden
- 404 = doesn't exist or wrong method
- 401 = needs authentication
- Map API surface by status code patterns

## Rate Limit Bypass

### IP Rotation
- X-Forwarded-For: 127.0.0.1, X-Forwarded: 1.2.3.4, etc.
- IPv6 rotation (64K addresses)
- AWS/GCP proxy rotation

### Parameter-Based Cache Hits
- `GET /api?t=1234567890` — unique `t` bypasses rate limit per parameter
- `GET /api?t=1234567890&t=9999999999` — array parameter
- Headers: `X-Request-ID: different-each-time`

### Timing Attacks
- Send requests in precise timing windows
- Exploit rate limit window reset boundaries
- Use concurrent connections before counter increments

## API Key/Token Discovery
- Check response bodies for leaked tokens
- Check debug endpoints: `/api/debug`, `/api/health`, `/api/status`
- Check `/.well-known/` endpoints
- JWT in URL vs Authorization header
- Token in cookie without HttpOnly