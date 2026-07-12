# Cloud Asset Discovery

## AWS Recon
S3 Bucket Discovery:
```
Buckets follow patterns: target-assets, target-backup, target-data, target-logs
```
Tools: s3scanner, cloud_enum, mass3

### Firebase Recon (Critical)
```bash
curl -s https://project.firebaseio.com/.json
curl -s https://project.firebaseio.com/users.json
```
Look for Firebase config in JS: apiKey, authDomain, databaseURL

## GCP Recon
- App Engine: *.appspot.com
- Cloud Functions: *.cloudfunctions.net
- Cloud Run: *.run.app

## Azure Recon
- Blob: *.blob.core.windows.net
- File: *.file.core.windows.net

## Multi-Cloud Enumeration
```bash
cloud_enum -k target -k target-dev -k target-prod
```

## Next Steps
1. Check bucket/file permissions (public read/write)
2. Inspect exposed file contents
3. Look for credentials and PII
4. Check for write access

---

> **Next**: [Automation & One-liners](07-automation-and-oneliners.md)
