# Mobile Security: 2026 Reference

## Android Testing

### Static Analysis
- **JADX**: Decompile APK to readable Java
- **apktool**: Decode resources, disassemble smali
- **MobSF**: Automated mobile security framework (static + dynamic)
- **Objection**: Runtime mobile exploration (Frida wrapper)

### Dynamic Analysis
- **Frida**: Hook running app — bypass SSL pinning, dump memory, modify return values
- **adb**: Android Debug Bridge — shell access, activity manager, intent fuzzing
- **Magisk**: Root for dynamic testing
- **Burp Suite**: Mobile proxy config

### Common Findings
1. Hardcoded API keys/tokens in decompiled source
2. Insecure data storage (SharedPreferences without encryption)
3. WebView with JS enabled + file:// access
4. Weak certificate pinning (okhttp3 bypass)
5. Exported activities without auth (adb am start -n ...)
6. Firebase database publicly writable

## iOS Testing

### Static Analysis
- **class-dump**: Extract Objective-C headers from Mach-O
- **Hopper/Ghidra**: Binary disassembly
- **MobSF**: iOS analysis as well
- **ipa install**: Sideload with iOS App Signer

### Dynamic Analysis
- **Frida**: iOS runtime manipulation (same as Android)
- **Objection**: Bypass jailbreak detection
- **Burp Suite**: Proxy via iOS VPN config

### Common Findings
1. Pasteboard leakage (sensitive data in system clipboard)
2. Insecure keychain storage (kSecAttrAccessible always)
3. UIPasteboard with no expiration
4. Insecure URL scheme handling (other apps can invoke)
5. Snapshot in app switcher shows sensitive data