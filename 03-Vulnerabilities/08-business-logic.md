# Business Logic Vulnerabilities: 2026 Reference

## Race Conditions (Turbo Intruder)

### Coupon/Referral Race
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=50, requestsPerConnection=100, pipeline=False)
    for i in range(100):
        engine.queue(target.req, [i])
```

- Apply same coupon 100x in parallel — each request deducts before balance check
- Referral bonus race: register 50 referrals simultaneously
- Money transfer race: send 50 withdrawal requests before balance deduction

### Price Manipulation
- Negative quantity: `quantity=-1` → negative total
- Integer overflow: `quantity=2147483648` → wraps to negative
- Fractional quantity: `quantity=0.5` → 50% discount
- Decimal precision: `0.01 * 1000 = 9.99` (floating point truncation)

## Cart/Order Tampering

### Addition/Removal Race
- Add expensive item to cart → complete checkout while item in cart
- Race to remove item after fraud check but before charge
- Manipulate cart while checkout processes authed total

### Price Override
```json
POST /api/cart
{"items": [{"id": 5, "price": 0.01, "quantity": 1}]}  // Client-side price accepted
```

### Quantity Integer Underflow
- `quantity=0` → negative revenue
- `quantity=-1` → vendor pays you
- Test ALL integer fields for overflow/underflow

## Feature Abuse

### Unlimited File Generation
- Export generates files without cleanup
- Coupon/referral code generation with no limit
- Automated account registration for bonus credits

### Step Skipping
- Jump to final checkout step without visiting cart
- Skip CAPTCHA/SMS verification by loading step_4.html directly
- Bypass KYC verification by calling enrollment complete API

### Time-Based Manipulation
- Subscription start/end date manipulation
- Free trial extending without payment
- Timezone-based billing cycle abuse

## Detection Approach
1. Read the application logic carefully — understand what SHOULD happen
2. Identify rate-limiting gaps (no limit on coupon use, no limit on referrals)
3. Test parallel requests with Turbo Intruder or Burp Intruder
4. Check order totals with negative/fractional values
5. Verify ALL authorization checks on EVERY step of a multi-step process