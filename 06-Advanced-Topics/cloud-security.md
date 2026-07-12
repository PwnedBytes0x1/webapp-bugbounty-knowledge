# Cloud Security Testing

## AWS Common Issues

### S3 Buckets
- Public bucket enumeration
- Data leakage
- Bucket policy misconfiguration

### IAM
- Over-permissive policies
- SSRF to metadata endpoint for credentials
- Unused roles and permissions

### Metadata Endpoint
```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

### S3 Discovery
```bash
s3scanner scan target.com
s3scanner scan -b bucket-names.txt
curl -s https://target-bucket.s3.amazonaws.com/
```

## GCP Common Issues

- Public Cloud Storage buckets
- Over-permissive IAM
- SSRF to metadata endpoints
- Firebase misconfigurations

### Metadata
```
http://metadata.google.internal/computeMetadata/v1/
```

## Azure Common Issues

- Blob storage public access
- SSRF to IMDS
- Key Vault misconfig
- Function Apps exposure

### IMDS
```
http://169.254.169.254/metadata/instance?api-version=2021-02-01
```
