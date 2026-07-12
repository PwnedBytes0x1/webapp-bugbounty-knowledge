# Cloud Security Testing

## AWS

### S3 Bucket Testing
```bash
# Bucket enumeration
aws s3 ls s3://target-backup --no-sign-request
aws s3 sync s3://target-backup ./ --no-sign-request
curl http://target-backup.s3.amazonaws.com/

# IAM enumeration
aws iam list-users
aws iam list-roles
aws iam list-policies
```

### EC2 / Lambda
```bash
# Instance metadata exploitation (SSRF)
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Lambda env variables (hardcoded secrets)
aws lambda get-function-configuration --function-name target-function
```

## GCP

### Storage Buckets
```bash
gsutil ls gs://target-bucket
gsutil cp gs://target-bucket/config.json .
```

### Cloud Functions
```bash
gcloud functions list
gcloud functions call target-function --data '{"test":true}'
```

## Azure

### Blob Storage
```bash
curl https://target.blob.core.windows.net/?comp=list
az storage blob list --account-name target --container-name prod
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| S3 public bucket read | 4.0 | Network, Low, No auth, Confidentiality low |
| Cloud metadata credential leak | 8.8 | Network, Low, No auth, Changed scope |
| Lambda env secret exposure | 7.5 | Network, Low, No auth |