# Mobile Application Testing

## Android Security Testing

### Setup
```bash
# Required tools
# Android Studio (emulator) or rooted device
# ADB (Android Debug Bridge)
# Frida (dynamic instrumentation)
# APKTool (reverse engineering)
# jadx (decompiler)

# Enable USB debugging on device
# Install APK
adb install app.apk
```

### Information Gathering
```bash
# Extract package info
adb shell dumpsys package com.target.app
adb shell pm path com.target.app

# Pull APK from device
adb pull /data/app/com.target.app/base.apk

# Decompile
apktool d app.apk -o decompiled/
jadx app.apk
```

### Common Mobile Vulnerabilities

#### Insecure Data Storage
- **SharedPreferences** — Unencrypted key-value storage
- **SQLite databases** — Unencrypted local DB
- **Internal/External storage** — File permissions
- **Keychain/Keystore** — Weak master password

```bash
# Check app data directory
adb shell run-as com.target.app
adb shell ls /data/data/com.target.app/
adb shell cat /data/data/com.target.app/shared_prefs/*.xml
```

#### Hardcoded Secrets
```bash
# Search decompiled source for secrets
grep -r "api_key\|apikey\|secret\|token\|password" decompiled/
grep -r "https\?://" decompiled/
```

#### Insecure Communication
- **No SSL pinning** — Intercept traffic with Burp Suite
- **Clear-text HTTP** — Traffic visible on network
- **WebView with JS enabled** — XSS attack surface

```bash
# Bypass SSL pinning with Frida
frida -U -f com.target.app --no-pause -l frida-ssl-pinning-bypass.js
```

#### Authentication/Authorization
- **Local authentication bypass** — Biometrics / PIN
- **OAuth token exposure** — Tokens in logs or storage
- **Session handling** — Tokens not invalidated on logout

### iOS Security Testing

#### Setup
- Mac with Xcode
- Jailbroken device or Frida
- objection (mobile exploration tool)

```bash
# Install IPA to device
ios-deploy -b app.ipa

# Decompile with class-dump
class-dump AppBinary -o headers/

# Bypass SSL pinning
objection -g com.target.app explore
ios sslpinning disable
```

### Mobile API Testing
- **API endpoints** — Find in decompiled code
- **Bypass root/jailbreak detection** — Frida scripts
- **Bypass certificate pinning** — Frida, objection
- **Replay attacks** — Capture and replay API calls

### Tools Summary

| Tool | Platform | Purpose |
|------|----------|---------|
| **APKTool** | Android | APK decompilation |
| **jadx** | Android | Java decompiler |
| **Frida** | Both | Dynamic instrumentation |
| **objection** | Both | Runtime mobile exploration |
| **MobSF** | Both | Mobile security framework |
| **Drozer** | Android | Security assessment |
| **Needle** | iOS | iOS security testing |

---

> **Next**: [Web3 Security](03-web3-security.md)
