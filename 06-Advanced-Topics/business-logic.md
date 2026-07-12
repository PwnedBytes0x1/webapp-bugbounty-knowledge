# Business Logic and Workflow Abuse

## What Are Business Logic Bugs?

Business logic vulnerabilities are flaws in the application's design that allow users to abuse intended functionality. Automated scanners cannot detect these.

## The 4-Question Framework

1. What is this feature supposed to do?
2. What happens if I skip a step?
3. What happens if I reverse the order?
4. What happens if I send unexpected values?

## High-Value Flows to Test

### E-Commerce
- Price manipulation (negative prices, currency conversion)
- Coupon/discount abuse (stacking, unlimited use)
- Quantity overflow
- Race conditions in checkout

### Financial
- Withdrawal without sufficient balance
- Double spending (race conditions)
- Rounding errors
- Fee manipulation

### User Management
- Rate limit bypass on account creation
- Email verification skip
- Mass assignment during registration
- Referral program abuse

## Race Condition Testing

Send multiple simultaneous requests:
```
POST /api/coupon/apply (simultaneous x3)
```

## Chaining Low-Severity Bugs

| Low Bug | Chain With | Result |
|---------|-----------|--------|
| Information Disclosure | IDOR | Account Takeover |
| Self-XSS | CSRF | Stored XSS on victim |
| Open Redirect | OAuth misconfig | Token theft |
| Missing Rate Limit | Auth Bypass | Credential Stuffing |
