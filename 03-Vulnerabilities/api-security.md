# API Security Testing

## OWASP API Security Top 10

| Rank | Risk |
|------|------|
| API1 | Broken Object Level Authorization (BOLA) |
| API2 | Broken Authentication |
| API3 | Broken Object Property Level Authorization |
| API4 | Unrestricted Resource Consumption |
| API5 | Broken Function Level Authorization (BFLA) |
| API6 | Unrestricted Access to Sensitive Business Flows |
| API7 | Server Side Request Forgery |
| API8 | Security Misconfiguration |
| API9 | Improper Inventory Management |
| API10 | Unsafe Consumption of APIs |

## REST API Testing Checklist

- Test IDOR on all object references
- Check HTTP methods (PUT/DELETE on read-only endpoints)
- Mass assignment via extra fields in JSON body
- Parameter pollution
- Content-type switching (JSON -> XML -> XXE)
- Rate limiting testing
- Excessive data exposure in responses
- Authentication bypass on API endpoints
- CORS misconfiguration

## GraphQL Testing

| Attack | Description |
|--------|-------------|
| Introspection | Map all types/queries via __schema |
| Batching Attack | Request multiple operations at once |
| Deep Recursion | Nested queries causing DoS |
| Direct Object Access | Bypass field-level authorization |
| Injection | SQLi/NoSQLi via GraphQL arguments |

## Tools
- Burp Suite
- Postman / Insomnia
- GraphQL Voyager
- Kiterunner (API route discovery)
