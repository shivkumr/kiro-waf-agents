---
name: waf-cost-optimization
description: AWS Well-Architected Cost Optimization pillar review criteria. Use when evaluating architecture for scaling efficiency, pricing models, data transfer, storage tiering, and resource waste.
---

# Cost Optimization — Review Criteria

## Checklist

### Right-Sizing & Scaling
- [ ] Compute sized to actual utilization (not peak + large buffer)
- [ ] Auto-scaling configured to scale down during off-hours
- [ ] Scheduled scaling for predictable patterns (business hours)
- [ ] Serverless for variable/unpredictable workloads
- [ ] Dev/test environments shut down outside business hours

### Pricing Models
- [ ] Savings Plans or Reserved Instances for steady-state workloads
- [ ] Spot instances for fault-tolerant batch/CI workloads
- [ ] Mix of commitment types matching workload predictability:
  - Compute SP: flexible across instance families
  - EC2 Instance SP: highest discount, specific family
  - RDS RI: specific engine and instance
- [ ] Commitment coverage reviewed quarterly

### Data Transfer
- [ ] VPC endpoints used (avoid NAT Gateway data charges)
- [ ] CloudFront for content delivery (cheaper egress)
- [ ] Same-AZ communication for chatty services where possible
- [ ] Cross-region replication justified by DR requirements
- [ ] S3 access patterns match storage class

### Storage Optimization
- [ ] S3 lifecycle policies (Standard → IA → Glacier → Delete)
- [ ] EBS volumes right-sized (gp3 > gp2, no unattached volumes)
- [ ] RDS storage auto-scaling enabled
- [ ] Old snapshots and AMIs cleaned up
- [ ] Log retention periods defined and enforced

### Governance & Visibility
- [ ] Cost allocation tags on all resources (Team, Service, Environment)
- [ ] AWS Budgets with alerts configured
- [ ] Cost anomaly detection enabled
- [ ] Regular cost reviews (weekly/monthly)
- [ ] Showback/chargeback model for shared services

### Unused Resource Detection
- [ ] Idle EC2 instances (CPU <5% sustained)
- [ ] Unattached EBS volumes
- [ ] Unused Elastic IPs
- [ ] Idle load balancers (no targets)
- [ ] Orphaned snapshots with no parent AMI/volume
- [ ] Inactive IAM users/roles

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Dev environments running 24/7 | 70% waste (only used ~8h/day) | Instance Scheduler or auto-scaling to 0 |
| No S3 lifecycle policies | Paying Standard prices for rarely-accessed data | Intelligent-Tiering or lifecycle rules |
| NAT Gateway for all traffic | $0.045/GB adds up fast | VPC endpoints for S3, DynamoDB, etc. |
| No commitment plans for stable workloads | Paying 40-60% more than necessary | Compute Savings Plans |
| Untagged resources | No visibility into who owns what | Enforce tagging via SCP + Config rules |
| Large data transfer cross-region for non-DR reasons | Unnecessary data transfer costs | Colocate services, use Global Accelerator |

## Key AWS Services

- **Cost Explorer**: Spend analysis, forecasting
- **Budgets**: Alerts and automated actions
- **Cost Anomaly Detection**: ML-based spend anomalies
- **Compute Optimizer**: Right-sizing recommendations
- **Savings Plans**: Flexible commitment discounts
- **S3 Intelligent-Tiering**: Automatic storage class optimization
- **Instance Scheduler**: Start/stop on schedule
- **Trusted Advisor**: Idle resource detection

## Quick Reference: Data Transfer Costs

| Path | Cost |
|------|------|
| Internet → AWS | Free |
| AWS → Internet | $0.09/GB (first 10TB) |
| Same AZ | Free (private IP) |
| Cross AZ | $0.01/GB each direction |
| Cross Region | $0.02/GB |
| NAT Gateway processing | $0.045/GB |
| VPC Endpoint (Gateway) | Free |
| VPC Endpoint (Interface) | $0.01/GB + hourly |
| CloudFront → Internet | $0.085/GB (cheaper than direct) |

## Example Findings

**Good**: "Production uses Compute Savings Plans (70% coverage), dev environments auto-stop at 7pm via Instance Scheduler, S3 lifecycle moves logs to Glacier after 30 days, all resources tagged with CostCenter and Team"

**Bad**: "12 m5.2xlarge instances running at 8% average CPU, no commitment plans, 500GB of gp2 EBS unattached, NAT Gateway processing 2TB/month to reach S3 (should use gateway endpoint)"
