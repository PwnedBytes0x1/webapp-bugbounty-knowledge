# Mobile Testing Methodology: 2026 Reference

## Setup Pipeline
1. Install app on rooted/jailbroken device (or emulator)
2. Configure Burp Suite as proxy (CA cert installed)
3. Bypass SSL pinning (Frida script: `frida -U -f com.app -l ssl_bypass.js --no-pause`)
4. Intercept and analyze traffic

## Key Checks

### API Communication
- All API calls from mobile → web API (likely vulnerable to same bugs)
- Custom headers, tokens, device IDs in requests
- Auth tokens stored in plaintext SharedPreferences
- Biometric authentication bypass (fallback to PIN)

### Local Storage
- SQLite databases in `/data/data/com.app/databases/`
- SharedPreferences XML in `/data/data/com.app/shared_prefs/`
- Realm databases
- Logcat output (adb logcat | grep -i password)

### Deep Link Handling
- Test all registered URL schemes
- Intent injection via malicious apps
- Deep link parameter pollution

## OWASP Mobile Top 10 (2024)
1. Improper Platform Usage
2. Insecure Data Storage
3. Insecure Communication
4. Insecure Authentication
5. Insufficient Cryptography
6. Insecure Authorization
7. Client Code Quality
8. Code Tampering
9. Reverse Engineering
10. Extraneous Functionality