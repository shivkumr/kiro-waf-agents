---
name: waf-cost-optimization
description: AWS Well-Architected Cost Optimization pillar. Use when reviewing, generating, or validating architecture for scaling efficiency, pricing models, data transfer, storage tiering, and resource waste.
---

# Cost Optimization

## Checklist (for reviewers/validators)

### Right-Sizing & Scaling
- [ ] Compute sized to actual utilization
- [ ] Auto-scaling configured to scale down during off-hours
- [ ] Scheduled scaling for predictable patterns
- [ ] Serverless for variable/unpredictable workloads
- [ ] Dev/test environments shut down outside business hours

### Pricing Models
- [ ] Savings Plans or Reserved Instances for steady-state workloads
- [ ] Spot instances for fault-tolerant batch/CI workloads
- [ ] Commitment coverage reviewed quarterly

### Data Transfer
- [ ] VPC endpoints used (avoid NAT Gateway data charges)
- [ ] CloudFront for content delivery (cheaper egress)
- [ ] Same-AZ communication for chatty services where possible
- [ ] Cross-region replication justified by DR requirements

### Storage Optimization
- [ ] S3 lifecycle policies configured
- [ ] EBS volumes right-sized (gp3 preferred over gp2)
- [ ] No unattached EBS volumes
- [ ] Old snapshots and AMIs cleaned up
- [ ] Log retention periods defined

### Governance & Visibility
- [ ] Cost allocation tags on all resources
- [ ] AWS Budgets with alerts configured
- [ ] Cost anomaly detection enabled
- [ ] Regular cost reviews scheduled

### Unused Resource Detection
- [ ] No idle EC2 instances (CPU <5% sustained)
- [ ] No unattached EBS volumes
- [ ] No unused Elastic IPs
- [ ] No idle load balancers (no targets)
- [ ] No orphaned snapshots

## Implementation Guide (for writers/generators)

### Right-Sizing Principles

- Start small, scale up based on metrics — never pre-provision for theoretical peak
- Use Fargate for containers (pay per vCPU-second, no idle waste)
- Lambda for event-driven (pay per invocation, zero cost at zero traffic)
- For steady-state EC2: choose Graviton instances (20% cheaper for same performance)
- Set ASG min to actual minimum needed, not "comfortable buffer"
- For dev/test: configure Instance Scheduler or ASG scheduled actions to stop outside 8am-6pm

### Data Transfer Awareness

- Use S3 Gateway Endpoint (free) instead of NAT Gateway ($0.045/GB) for S3 access
- Use DynamoDB Gateway Endpoint (free) for DynamoDB access
- Place chatty services in the same AZ when possible (free same-AZ private IP traffic)
- Use CloudFront even for same-region delivery (egress through CF is cheaper than direct)
- Avoid cross-region data transfer unless required for DR — document justification
- If cross-AZ traffic is significant: consider AZ-affinity patterns for chatty services

### Storage Tiering

- S3: apply Intelligent-Tiering for unknown access patterns, or explicit lifecycle rules:
  - Standard → Infrequent Access after 30 days
  - IA → Glacier Instant Retrieval after 90 days
  - Glacier → Deep Archive after 180 days (or delete)
- EBS: use gp3 (not gp2) — 20% cheaper with better baseline performance
- Set explicit log retention: 7 days dev, 30 days staging, 90 days prod
- Enable S3 lifecycle to expire incomplete multipart uploads after 7 days
- Schedule monthly cleanup of unattached volumes and orphaned snapshots

### Tagging Strategy

- Mandatory tags on every resource: `Environment`, `Service`, `Team`, `CostCenter`
- Enforce via SCP or Config rules that deny untagged resource creation
- Use tag-based cost allocation reports in Cost Explorer
- Add `AutoStop` tag for resources that should be scheduled on/off

### Commitment Planning

- Compute Savings Plans: cover steady-state baseline (start at 50-60% of on-demand)
- Use 1-year no-upfront for flexibility, 3-year all-upfront only for proven stable workloads
- Review coverage quarterly — adjust as workload patterns change
- Never commit more than current steady-state floor

### Diagram Components

When generating architecture diagrams:
- Annotate NAT Gateways with "consider VPC endpoint" where applicable
- Show VPC endpoints for S3 and DynamoDB (highlight cost savings)
- Label compute resources with sizing tier (not specific instance type in diagrams)
- Show scheduled scaling indicators on dev/test resources
- Note where CloudFront reduces origin egress costs

## Data Transfer Cost Reference

| Path | Cost |
|------|------|
| Internet → AWS | Free |
| AWS → Internet | ~$0.09/GB (first 10TB) |
| Same AZ (private IP) | Free |
| Cross AZ | $0.01/GB each direction |
| Cross Region | $0.02/GB |
| NAT Gateway processing | $0.045/GB |
| VPC Endpoint (Gateway) | Free |
| VPC Endpoint (Interface) | $0.01/GB + hourly |
| CloudFront → Internet | ~$0.085/GB (cheaper than direct) |

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Dev environments running 24/7 | 70% waste | Instance Scheduler or scale-to-zero |
| No S3 lifecycle policies | Paying Standard for cold data | Intelligent-Tiering or lifecycle rules |
| NAT Gateway for S3/DynamoDB traffic | $0.045/GB wasted | Gateway endpoints (free) |
| No commitment plans for stable workloads | Paying 40-60% more | Compute Savings Plans |
| Untagged resources | No cost visibility | Enforce tagging via SCP + Config |
| gp2 EBS volumes | 20% more expensive than gp3 | Migrate to gp3 |

## Key AWS Services

- **Cost Explorer**: Spend analysis, forecasting
- **Budgets**: Alerts and automated actions
- **Cost Anomaly Detection**: ML-based spend anomalies
- **Compute Optimizer**: Right-sizing recommendations
- **Savings Plans**: Flexible commitment discounts
- **S3 Intelligent-Tiering**: Automatic storage optimization
- **Instance Scheduler**: Start/stop on schedule
- **Trusted Advisor**: Idle resource detection
