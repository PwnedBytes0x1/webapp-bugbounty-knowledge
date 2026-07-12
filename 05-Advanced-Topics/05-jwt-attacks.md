# JWT Attacks: 2026 Reference

## Algorithm Confusion

### "none" Algorithm
- `alg: none` — some libraries accept unsigned tokens
- `alg: None` (capital), `alg: NONE`
- `alg: nOnE`

### RS256 → HS256
- Change `alg` from `RS256` to `HS256`
- Use RSA public key (often available) as HMAC secret
- Common root: `"-----BEGIN PUBLIC KEY-----"` hardcoded in app

### JWK Injection
- Include `"jwk": {...}` header with attacker's public key
- Library decodes and trusts inline JWK
- Server creates symmetric key from attacker's public key

### KID Injection
- `"kid": "../../../../dev/null"` — reads empty file as key
- `"kid": "0000"` — SQLi if KID looked up in DB
- `"kid": "|| cmd"` — command injection in KID lookup

## Weak Key Cracking
```bash
hashcat -m 16500 jwt.txt rockyou.txt
jwt_tool jwt.txt -C -d wordlist.txt
jwt_tool jwt.txt -K -k public.key  # RS to HS confusion
```

## Claims Manipulation
- Change `sub` to another user ID
- Remove `exp` entirely
- Set `iat` far in the past
- Add `admin: true` or `role: admin`

## JWT Tooling
- **jwt_tool**: Cracking, scanning, forging
- **jwt.io**: Debug/verify tokens
- **Burp JWT Editor**: Modify JWTs in proxy

## Defensive Checklist
1. Always validate `alg` against allowlist (never accept `none`)
2. Use separate validation code paths for symmetric vs asymmetric
3. Never trust `jwk`, `jku`, `kid` from untrusted sources
4. Validate KID against allowlist (no path traversal, no SQLi)
5. Enforce token expiration server-side (not just verify `exp`)
6. Rotate signing keys regularly