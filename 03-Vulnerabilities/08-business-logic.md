# Business Logic Vulnerabilities

Business logic vulnerabilities are flaws in the design and implementation of an application's business rules. Unlike technical vulnerabilities (SQLi, XSS), these require understanding the application's intended workflow.

## Why Business Logic Bugs Pay Well

- **Unique to each application** — No scanner can find them
- **High impact** — Can lead to financial loss, data manipulation, privilege escalation
- **Hard to prevent** — Require deep understanding of business requirements

## Common Types

### 1. Coupon/Discount Abuse
```python
# Reuse coupon multiple times
POST /apply-coupon
{"coupon_code": "WELCOME20"}

# Stack coupons
POST /apply-coupon
{"coupon_code": "SAVE10"}
POST /apply-coupon
{"coupon_code": "FREESHIP"}

# Apply negative quantities
POST /cart/update
{"item_id": 1, "quantity": -100}

# Currency conversion rounding
POST /checkout
{"currency": "USD", "amount": 0.01}
```

### 2. Race Conditions (TOCTOU)
- **Gift card balance** — Use same gift card on multiple purchases simultaneously
- **Wallet withdrawal** — Multiple simultaneous withdrawal requests
- **Voting/liking** — Like a post unlimited times via parallel requests

```javascript
// Send parallel requests
const requests = [];
for (let i = 0; i < 100; i++) {
    requests.push(fetch("/api/wallet/withdraw", {
        method: "POST",
        body: JSON.stringify({amount: 100})
    }));
}
await Promise.all(requests);
```

### 3. Account Takeover via Logic
- **Email change without verification** — Change email on account X, use forgot password on new email
- **Account linking** — Link attacker's social account to victim's account
- **Password reset flow** — Skip steps in multi-step reset process

### 4. Transaction Manipulation
- **Price manipulation** — Change price parameter during checkout
- **Fee avoidance** — Avoid transaction fees by manipulating parameters
- **Currency arbitrage** — Exploit exchange rate conversion timing

### 5. Privilege Escalation via Workflow
- **Self-signup to admin** — Exploit registration flow to assign admin role
- **Skip subscription payment** — Sign up for trial, manipulate account status
- **Invite system abuse** — Invite self to organization with elevated role

### 6. Bypassing Restrictions
- **Rate limit bypass** — Rotate IPs, headers, user agents
- **Content restriction** — View age-restricted / geo-restricted content
- **Feature gating** — Access premium features without subscription

## Testing Methodology

1. **Understand the business flow** — Create accounts, make purchases, try all features
2. **Map state transitions** — Identify all possible paths through the workflow
3. **Test edge cases** — Negative numbers, zero values, max integers, special characters
4. **Manipulate parameters** — Change prices, quantities, user IDs, roles
5. **Test concurrency** — Send parallel requests for race conditions
6. **Chain features** — Combine features in unexpected ways

## Real-World Examples

- **Twitter** — Change username to include `admin` for internal tool access
- **Uber** — Manipulate surge pricing parameters
- **PayPal** — Transaction ID manipulation for double spending
- **Amazon** — Price manipulation via parameter injection

## Prevention

1. **Server-side validation** for all business-critical operations
2. **Idempotency keys** — Prevent duplicate operations
3. **Atomic transactions** — Prevent race conditions with database locks
4. **State machine enforcement** — Only allow valid state transitions
5. **Rate limiting** — Per-user and per-IP
6. **Audit trails** — Log all business-critical operations
7. **Unit tests for business rules** — Test edge cases

---

> **Next**: [API Security](09-api-security.md)
