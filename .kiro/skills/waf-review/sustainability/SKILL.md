---
name: waf-sustainability
description: AWS Well-Architected Sustainability pillar review criteria. Use when evaluating architecture for efficient resource utilization, managed services, data lifecycle, and carbon-aware design.
---

# Sustainability — Review Criteria

## Checklist

### Efficient Resource Utilization
- [ ] Right-sized instances (no sustained <20% CPU utilization)
- [ ] Auto-scaling matches demand (scale down during quiet periods)
- [ ] Serverless for bursty/variable workloads (zero compute when idle)
- [ ] Graviton processors used (up to 60% less energy for same performance)
- [ ] Spot instances for batch (reuse spare capacity)

### Managed Services Over Self-Managed
- [ ] Managed database (RDS/Aurora) instead of DB on EC2
- [ ] Managed containers (Fargate) instead of self-managed ECS/EC2
- [ ] Managed Kafka (MSK Serverless) instead of self-managed brokers
- [ ] SES/SNS instead of self-hosted email/notification servers
- [ ] Managed services share resources across customers = higher utilization

### Data Lifecycle & Storage
- [ ] Data retention policies defined (don't store forever)
- [ ] S3 lifecycle transitions (IA → Glacier → Delete)
- [ ] Log retention periods set (not indefinite)
- [ ] Old snapshots, AMIs, and artifacts cleaned up
- [ ] Compression enabled for stored and transferred data

### Region & Network
- [ ] Region selection considers carbon intensity where requirements allow
- [ ] Data transfer minimized (colocate compute and storage)
- [ ] Caching reduces redundant computation and data transfer
- [ ] CDN reduces origin requests (less backend processing)

### Software & Architecture Patterns
- [ ] Efficient algorithms and queries (reduce compute time)
- [ ] Async processing (batch work for higher throughput per watt)
- [ ] Event-driven (no idle polling)
- [ ] Efficient data formats (Parquet > CSV for analytics)

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Over-provisioned instances running idle | Wasted energy, unnecessary carbon | Right-size, auto-scale, go serverless |
| Self-managed services on EC2 | Lower utilization than shared managed services | Migrate to managed (RDS, Fargate, MSK) |
| No data lifecycle — store everything forever | Unnecessary storage energy | Lifecycle policies, retention limits |
| Polling-based architecture | Constant compute for nothing | Event-driven with EventBridge/SQS |
| x86 when Graviton compatible | Missing 60% energy efficiency gain | Test and migrate to Graviton (arm64) |
| Redundant data copies with no cleanup | Storage waste | Deduplication, lifecycle, TTL |

## Key AWS Services

- **Graviton**: ARM-based, energy-efficient processors
- **Lambda / Fargate**: Zero compute when idle
- **S3 Intelligent-Tiering**: Auto-optimizes storage
- **Auto Scaling**: Match demand, eliminate waste
- **Customer Carbon Footprint Tool**: Track emissions
- **Compute Optimizer**: Find over-provisioned resources

## AWS Regions — Carbon Intensity (Lower = Better)

Regions powered by renewable energy:
- **us-west-2 (Oregon)**: Hydropower, one of the lowest carbon regions
- **eu-west-1 (Ireland)**: High renewable mix
- **ca-central-1 (Canada)**: Hydropower
- **eu-north-1 (Stockholm)**: Near 100% renewable

Note: Region selection should prioritize latency and compliance requirements first; sustainability is a tiebreaker.

## Example Findings

**Good**: "Fargate Spot for batch processing, Graviton instances for all ECS tasks, S3 lifecycle deletes logs after 90 days, CloudFront reduces 80% of origin traffic, Customer Carbon Footprint Tool reviewed quarterly"

**Bad**: "Fleet of 20 m5.large instances running 24/7 at 12% average utilization, self-managed RabbitMQ on EC2, CloudWatch logs set to never expire (300TB accumulated), no Graviton adoption despite compatible workloads"
