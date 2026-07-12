# Mobile Application Security

## Android Testing

### APK Analysis
```bash
# Decompile APK
apktool d target.apk
jadx-gui target.apk

# Extract manifest
aapt dump badging target.apk

# Check for debug mode
# AndroidManifest.xml: android:debuggable="true"
```

### Common Issues
- Hardcoded API keys / tokens in source
- Insecure WebView (JavaScript enabled, file access)
- Weak SSL pinning (bypass with Frida)
- Exported activities/components
- SQLite database with plaintext data
- Insecure data storage (SharedPreferences)
- Deep link hijacking
- Tapjacking

### Frida Scripts
```javascript
// Bypass SSL pinning
setTimeout(function() {
    Java.perform(function() {
        var SSLPinning = Java.use("okhttp3.CertificatePinner");
        SSLPinning.check.overload('java.lang.String', 'java.util.List').implementation = function() {
            return;
        };
    });
}, 0);
```

### Burp Suite Proxy
```bash
# Android emulator
adb shell settings put global http_proxy 10.0.2.2:8080

# Install Burp CA
adb push burp-ca.cer /sdcard/
adb shell am start -a android.intent.action.VIEW -d "content://com.android.externalstorage.documents/document/primary%3Aburp-ca.cer"
```

## iOS Testing

### IPA Analysis
```bash
# Decrypt IPA (jailbroken)
frida-ios-dump target.ipa

# Check Info.plist
plutil -p Info.plist
```

### Common Issues
- Insecure data storage (NSUserDefaults, Keychain)
- URL scheme hijacking
- Pasteboard data leakage
- Background screenshot (Snap prevention bypass)

### Objection Toolkit
```bash
objection patchapk -s target.apk
objection explore -P target.apk
```