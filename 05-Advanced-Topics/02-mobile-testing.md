# Mobile Application Security Testing

## Android Static Analysis

```bash
jadx-gui target.apk
apktool d target.apk -o target_decoded/
cat target_decoded/AndroidManifest.xml
```

### Key Findings
- Hardcoded secrets: `grep -r "api_key|password|secret|token" target_decoded/`
- Exported components: `android:exported="true"`
- WebView vulns: `setJavaScriptEnabled(true)`, `addJavascriptInterface()`
- Insecure storage: plaintext in SharedPreferences, SQLite
- Deep links without validation

## Android Dynamic Analysis

```bash
# SSL Pinning Bypass
objection -g com.target.app explore
android sslpinning disable

# Frida Runtime Manipulation
frida -U -l bypass_root.js com.target.app

# Burp Proxy
adb shell settings put global http_proxy 192.168.1.100:8080
```

### Common Vulns
1. Root detection bypass -> weak client-side control
2. Insecure data storage (tokens in SharedPreferences)
3. Missing SSL pinning -> MITM
4. Deep link hijacking -> intent interception
5. WebView JS bridge -> XSS to RCE
6. Exported Content Providers -> SQLi, data leak

## iOS Testing

```bash
# Dump decrypted IPA
frida-ios-dump -u root -H 192.168.1.100 "Target App"

# Class dump (Obj-C)
class-dump -H TargetApp.app -o headers/

# Objection
objection --gadget "Target App" explore
```

### iOS-Specific Findings
- Keychain: kSecAttrAccessibleAlways = weak
- ATS bypass: NSAllowsArbitraryLoads = true
- URL scheme hijacking
- Pasteboard leakage

## Mobile API Testing

```bash
# Extract API endpoints from decompiled app
grep -rohP 'https?://[^"'"' ]+' target_decoded/ | sort -u
```

## Tooling
```bash
docker run -p 8000:8000 opensecurity/mobile-security-framework-mobsf
pip install frida-tools
pip install objection
```
