---
name: waf-security
description: AWS Well-Architected Security pillar. Use when reviewing, generating, or validating architecture for IAM, network isolation, encryption, threat detection, and secrets management.
---

# Security

## Checklist (for reviewers/validators)

### Identity & Access Management
- [ ] Least privilege IAM policies (no `*` actions or resources)
- [ ] No long-lived access keys; use IAM roles and instance profiles
- [ ] MFA enforced for human users
- [ ] Service-to-service auth via IAM roles (not shared secrets)
- [ ] SCPs applied at Organization level for guardrails
- [ ] Regular access reviews and credential rotation

### Network Security
- [ ] VPC with private and public subnet separation
- [ ] Security groups: deny by default, minimal ingress rules
- [ ] NACLs for subnet-level defense-in-depth
- [ ] No direct internet access for backend services
- [ ] VPC endpoints for AWS service access (no internet path)
- [ ] Network Firewall or third-party IDS/IPS for inspection

### Data Protection
- [ ] Encryption at rest (KMS-managed keys for S3, RDS, EBS, DynamoDB)
- [ ] Encryption in transit (TLS 1.2+ everywhere)
- [ ] S3 Block Public Access enabled at account level
- [ ] Database connections via TLS
- [ ] Backup encryption enabled

### Secrets Management
- [ ] Secrets in Secrets Manager or Parameter Store (SecureString)
- [ ] No hardcoded credentials in code, configs, or environment variables
- [ ] Automatic rotation configured
- [ ] Secrets referenced by ARN, not value

### Detection & Response
- [ ] GuardDuty enabled in all regions
- [ ] Security Hub with compliance standards enabled
- [ ] CloudTrail enabled with log file validation
- [ ] VPC Flow Logs enabled
- [ ] AWS Config rules for continuous compliance

### Edge Protection
- [ ] WAF on ALB/CloudFront with rate limiting
- [ ] Shield Advanced for DDoS protection (if critical)
- [ ] Bot control rules for public-facing APIs

## Implementation Guide (for writers/generators)

### IAM

- Create one IAM role per service/function — never share roles across services
- Scope policies to specific resource ARNs, not `*`
- Use conditions (aws:SourceVpc, aws:PrincipalOrgID) to further restrict
- For Lambda/ECS: create a task execution role (pulls images, writes logs) AND a task role (app permissions) — keep them separate
- Never output access keys in Terraform — use instance profiles and IRSA (EKS)
- Add `deny` statements in SCPs for critical actions (leaving org, disabling CloudTrail)

### Network Isolation

- Place compute and databases in private subnets (no internet gateway route)
- Public subnets only for ALB and NAT Gateways
- One security group per tier (web, app, db) — reference other SGs by ID for inter-tier rules
- Default: deny all ingress, allow only specific ports between tiers
- Use VPC endpoints (gateway type for S3/DynamoDB = free, interface type for other services)
- Enable VPC Flow Logs to CloudWatch Logs (minimum: REJECT only for cost efficiency)
- If multi-account: use PrivateLink or Transit Gateway, never VPC peering sprawl

### Encryption

- Every data store MUST have encryption enabled — no exceptions
- Use dedicated KMS CMK per service or per sensitivity level (not default aws/service key)
- Enable automatic key rotation on all CMKs (1 year)
- Enforce TLS on all connections: RDS `require_ssl`, ElastiCache `transit_encryption_enabled`, ALB HTTPS listeners only
- S3: enable default encryption (SSE-KMS) + bucket policy denying non-TLS requests
- EBS: enable default encryption at account level via EC2 settings

### Secrets

- Store all credentials in Secrets Manager with automatic rotation
- Reference secrets by ARN in ECS task definitions, Lambda environment, etc.
- Never put secrets in Terraform variables or tfvars files — use data sources to read from Secrets Manager
- Set rotation schedule: 30 days for database credentials, 90 days for API keys
- Use Secrets Manager VPC endpoint so secrets never traverse the internet

### Detection

- Enable GuardDuty in all active regions — aggregate to a security account
- Enable Security Hub with CIS AWS Foundations and AWS Foundational Security Best Practices standards
- CloudTrail: multi-region, organization trail, log file validation enabled, logs to encrypted S3
- Set up Config rules for: encrypted-volumes, s3-bucket-public-read-prohibited, restricted-ssh

### Edge Protection

- Attach WAF to every public-facing ALB and CloudFront distribution
- Use AWS Managed Rule Groups as baseline (Core Rule Set, SQL injection, Known Bad Inputs)
- Add rate-based rule (2000 requests/5min per IP as starting point)
- For APIs: add API Gateway resource policy restricting to known CIDR ranges or VPC

### Diagram Components

When generating architecture diagrams:
- Show WAF/Shield in front of ALB/CloudFront
- Draw clear VPC boundary with public/private subnet separation
- Show security group boundaries around each tier
- Include KMS key icons connected to encrypted data stores
- Show CloudTrail/GuardDuty/Security Hub in a "security services" group
- Show VPC endpoints where services access S3/DynamoDB

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| `Action: "*", Resource: "*"` in IAM | Full account compromise if creds leak | Scope to specific actions and resources |
| Security groups with 0.0.0.0/0 ingress | Attack surface exposure | Restrict to known CIDR ranges or SG refs |
| S3 buckets without Block Public Access | Data breach | Account-level S3 Block Public Access |
| Hardcoded DB password in env vars | Credential exposure in console/logs | Secrets Manager with rotation |
| No GuardDuty or Security Hub | Blind to threats | Enable in all regions |
| No WAF on public ALB | Vulnerable to OWASP Top 10 | AWS WAF with managed rule groups |

## Key AWS Services

- **IAM**: Roles, Policies, Identity Center (SSO)
- **KMS**: Key management, automatic rotation
- **Secrets Manager**: Secret storage, auto-rotation
- **GuardDuty**: Threat detection
- **Security Hub**: Posture management, compliance
- **WAF**: Web application firewall
- **Shield**: DDoS protection
- **CloudTrail**: API audit logging
- **VPC**: Network isolation, endpoints, flow logs
- **Config**: Compliance rules, remediation
