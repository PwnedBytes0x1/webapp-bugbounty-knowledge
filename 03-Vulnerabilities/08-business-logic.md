# Business Logic Flaws: The High-Payout Territory

## Why Business Logic Pays More

Automated scanners cannot detect business logic flaws because they require understanding the intended behavior. These bugs consistently pay $5,000–$50,000+ because:
1. They are unique — no duplicate problem vs other hunters
2. They demonstrate financial impact directly
3. They require human creativity to find — programs value this

## Common Business Logic Classes

### Currency Manipulation
```http
POST /api/cart/add
{"item_id": 123, "quantity": 1, "price": 0.01, "currency": "USD"}
# If the server trusts client-side price, create arbitrary discounts
# Test negative quantities (item = item + (-1) → credit?)
# Test fractional quantities (0.001 → rounding issues)
```

### Coupon/Discount Abuse
```bash
# Test coupon stackability
POST /api/cart/apply-coupon
{"code": "FIRST10"}
POST /api/cart/apply-coupon
{"code": "VIP20"}
# Does the server enforce single-coupon or allow stacking?
```

### Race Conditions
```http
# Send 10 concurrent requests for the same limited offer
POST /api/redeem
{"code": "LIMITED_OFFER"}
# If no atomic check, all 10 succeed
# Checkout race: add item, change cart, submit payment simultaneously
```

### Multi-Step Flow Tampering
```bash
# Step 1: select shipping address
# Step 2: select payment method
# Step 3: confirm order
# Test: go directly to step 3 with modified parameters
# Test: complete step 1 with low price, modify price request in step 2
# Test: replay step 2 from a different session
```

### Account Registration Abuse
```bash
# Same email, different case → duplicate accounts?
user@test.com vs User@test.com
# Invite code reuse
# Referral program abuse (refer yourself)
# Account deletion → re-register with same email (data not fully deleted)
```

### Escalation Paths

Business logic flaws often chain:
1. Quantity manipulation during checkout
2. Apply coupon on manipulated price
3. Use referral credit for further discount
4. Result: $0 purchase of $1,000 item → Critical financial impact

## Hunting Methodology

1. Understand the intended flow by reading docs or observing normal behavior
2. Enumerate every step in the flow
3. Test each step individually with modified parameters
4. Test reordering steps
5. Test replaying individual steps
6. Test concurrent execution (race conditions)
7. Test with boundary values (0, negative, max int, empty strings)
8. Document the intended vs actual behavior
