# Cloud Security: 2026 Reference

## Attack Surface by Provider

### AWS
- **S3 bucket enumeration**: Common names, permission check
- **IAM privilege escalation**: Permission chains
- **Lambda function takeover**: Unused env variables, old runtimes
- **CloudFront misconfig**: Origin access, CNAME takeover
- **EC2 SSRF**: IMDSv1/v2, user-data, instance metadata
- **ECS task definitions**: Secret env vars accessible if permissive
- **Route53 subdomain takeover**: Orphaned DNS records

### GCP
- **Cloud Storage bucket enum**: Similar to S3 pattern
- **IAM role misconfig**: Primitive escalation
- **Cloud Functions**: Old runtimes, env vars
- **Firebase config**: API key exposure, database perms
- **Kubernetes Engine**: Public dashboards, RBAC gaps

### Azure
- **Blob Storage**: Anonymous access, misconfigured containers
- **App Service**: Old runtimes, auto-generated domains
- **Key Vault**: Misconfigured access policies
- **Azure Functions**: Env var leaks, permissive triggers.

## Cloud Metadata SSRF
Ref: SSRF section for full metadata endpoint reference.

## Tools
- ScoutSuite, Pacu (AWS), GCPBucketBrute, CloudSploit, MicroBurst (Azure)