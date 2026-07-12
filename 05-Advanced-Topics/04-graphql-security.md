# GraphQL Security: 2026 Reference

## Attack Surface

### Introspection
- `/graphql?query={__schema{types{name}}}`
- Disabled but field-guess still works
- Batch queries bypass single-query rate limits

### Batching Attacks
```graphql
query {
  a: user(id: 1) { creditCard }
  b: user(id: 2) { creditCard }
  c: user(id: 3) { creditCard }
}
```

### Mutation Enumeration
- Brute force credentials via login mutation
- Reset password: username + newPassword
- Mass assignment

### Depth-Based DoS
- Deeply nested queries crash parser
- `query { a: posts { comments { user { posts { comments { ... } } } } } }`
- Circular fragments: `fragment X on Y { ...X }`

## Tools
- **InQL** (Burp extension): Introspection + query generation
- **graphql-path-enum**: Brute-force field names (introspection disabled)
- **Clairvoyance**: Introspection data reconstruction (blind)

## Defensive Checklist
1. Disable introspection in production
2. Set max query depth (5-7 recommended)
3. Set max query cost (rate limit by complexity)
4. Implement query allowlisting
5. Add rate limiting per user/IP
6. Validate all mutation inputs
7. Enforce auth on each resolver (not at endpoint level)