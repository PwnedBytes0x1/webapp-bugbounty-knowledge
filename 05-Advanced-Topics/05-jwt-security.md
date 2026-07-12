# JWT Security: Concise Reference

## Critical Checks
- **Algorithm**: Reject `none`, validate expected `alg` only
- **Key**: Don't confuse RS/HS, validate `kid` origins
- **Expiration**: Enforce server-side, not just verify
- **Signature**: Validate before processing claims
- **Claims**: Whitelist expected claims, reject unknown
- **Storage**: localStorage is XSS-accessible; httpOnly cookies preferred

## Common CVEs
- CVE-2015-9235 (none algorithm)
- CVE-2016-5431 (RS→HS confusion)
- CVE-2018-0114 (kid injection)
- CVE-2024-XXXX (jku open redirect)