# Insecure Direct Object References (IDOR)

## What is IDOR?

IDOR occurs when an application exposes direct references to internal objects and fails to verify authorization.

## Common Locations

- REST API endpoints: /api/users/12345
- File downloads: /download?file=report_101.pdf
- Invoices: /invoice/INV-2026-001
- User profiles: /profile?user_id=42

## Testing Methodology

### 1. Create Two Accounts
Register as victim and attacker

### 2. Identify Object References
Look for numeric IDs, UUIDs, hashes in:
- URL paths and parameters
- JSON request/response bodies
- Hidden form fields
- Cookies

### 3. Test Horizontal IDOR
GET /api/orders/12345 -> GET /api/orders/12346

### 4. Test Vertical IDOR
Access admin functions with user token

## Escalation Paths

- Read IDOR (PII): Automate to show scale
- Write IDOR (email/password): Direct ATO
- UUID-based IDOR: Find UUID leak elsewhere

## Prevention
- Server-side authorization checks on every request
- Use indirect reference maps
- Never rely on UI-only restrictions
- Test API endpoints independently from UI
