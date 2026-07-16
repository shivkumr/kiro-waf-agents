---
name: waf-sustainability
description: AWS Well-Architected Sustainability pillar. Use when reviewing, generating, or validating architecture for efficient resource utilization, managed services, data lifecycle, and carbon-aware design.
---

# Sustainability

## Checklist (for reviewers/validators)

### Efficient Resource Utilization
- [ ] Right-sized instances (no sustained <20% CPU utilization)
- [ ] Auto-scaling matches demand (scale down during quiet periods)
- [ ] Serverless for bursty/variable workloads (zero compute when idle)
- [ ] Graviton processors used where compatible
- [ ] Spot instances for batch workloads

### Managed Services Over Self-Managed
- [ ] Managed database (RDS/Aurora) instead of DB on EC2
- [ ] Managed containers (Fargate) instead of self-managed ECS/EC2
- [ ] Managed streaming (MSK Serverless) instead of self-managed brokers
- [ ] SES/SNS instead of self-hosted email/notification servers

### Data Lifecycle & Storage
- [ ] Data retention policies defined
- [ ] S3 lifecycle transitions configured
- [ ] Log retention periods set (not indefinite)
- [ ] Old snapshots and artifacts cleaned up
- [ ] Compression enabled for stored and transferred data

### Region & Network
- [ ] Region selection considers carbon intensity where requirements allow
- [ ] Data transfer minimized (colocate compute and storage)
- [ ] Caching reduces redundant computation and transfer
- [ ] CDN reduces origin requests

### Software & Architecture Patterns
- [ ] Efficient algorithms (reduce compute time)
- [ ] Async processing (batch for higher throughput per watt)
- [ ] Event-driven (no idle polling)
- [ ] Efficient data formats (Parquet over CSV for analytics)

## Implementation Guide (for writers/generators)

### Resource Efficiency

- Default to Graviton (arm64) for all new workloads — up to 60% less energy per unit of compute
- Use Fargate or Lambda for workloads that don't need 24/7 compute — zero energy at zero traffic
- Set auto-scaling policies that aggressively scale down (don't leave warm pools over-sized)
- For batch jobs: use Spot Instances (reuses spare capacity that would otherwise be idle)
- Right-size from day one — start at the smallest viable instance, scale up only with evidence
- Review Compute Optimizer monthly and action recommendations within 30 days

### Managed Services Priority

- Always prefer managed over self-managed:
  - RDS/Aurora over PostgreSQL on EC2
  - Fargate over self-managed ECS on EC2
  - MSK Serverless over Kafka on EC2
  - OpenSearch Service over Elasticsearch on EC2
  - ElastiCache over Redis on EC2
- Reason: managed services share infrastructure across customers = higher overall utilization
- Only use self-managed when a specific feature or version requires it — document justification

### Data Lifecycle

- Define retention policy for every data store at creation time — never "keep forever" as default
- S3: apply lifecycle rules immediately, don't add later:
  - Move to IA after 30 days
  - Move to Glacier after 90 days
  - Delete or Deep Archive after 365 days (unless compliance requires longer)
- CloudWatch Logs: set retention to minimum needed (7 days dev, 30 staging, 90 prod)
- Enable compression on all log ingestion and data transfers
- Use columnar formats (Parquet, ORC) for analytics instead of CSV/JSON
- Clean up: schedule monthly deletion of unneeded snapshots, old AMIs, abandoned artifacts

### Region Selection

- When latency and compliance requirements allow a choice: prefer low-carbon regions
  - us-west-2 (Oregon): hydropower
  - eu-west-1 (Ireland): high renewable mix
  - eu-north-1 (Stockholm): near 100% renewable
  - ca-central-1 (Canada): hydropower
- Document region selection rationale (latency/compliance first, sustainability as tiebreaker)
- Use Customer Carbon Footprint Tool in AWS Billing console to track and report

### Architecture Patterns

- Event-driven over polling: use EventBridge triggers, not cron-based polling loops
- Batch processing: aggregate work items and process in bulk (higher throughput per joule)
- Cache aggressively: every cache hit = avoided computation + avoided network transfer
- Use efficient serialization: Protocol Buffers or MessagePack over JSON for high-volume internal APIs
- Design for right-sized responses: paginate, filter at server, avoid over-fetching

### Diagram Components

When generating architecture diagrams:
- Label Graviton resources explicitly
- Show serverless components (Lambda, Fargate) with "scales to zero" annotation
- Show caching layers as reducing load arrows from origin
- Include S3 lifecycle annotations (tiering indicators)
- Note managed services vs self-managed where relevant
- If multi-region: annotate primary region choice rationale

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Over-provisioned instances running idle | Wasted energy | Right-size, auto-scale, go serverless |
| Self-managed services on EC2 | Lower utilization | Migrate to managed services |
| No data lifecycle — store everything forever | Unnecessary storage | Lifecycle policies, retention limits |
| Polling-based architecture | Constant compute for nothing | Event-driven with EventBridge/SQS |
| x86 when Graviton-compatible | Missing 60% energy efficiency gain | Test and migrate to arm64 |
| CSV/JSON for large analytics datasets | Wasted storage and compute | Columnar formats (Parquet) |

## AWS Regions — Carbon Intensity (Lower = Better)

| Region | Power Source | Notes |
|--------|-------------|-------|
| us-west-2 (Oregon) | Hydropower | One of the lowest carbon regions |
| eu-north-1 (Stockholm) | Near 100% renewable | Excellent for EU workloads |
| eu-west-1 (Ireland) | High renewable mix | Good EU option |
| ca-central-1 (Canada) | Hydropower | Good for North America |

Note: Always prioritize latency and compliance requirements. Sustainability is a tiebreaker when multiple regions meet functional requirements.

## Key AWS Services

- **Graviton**: ARM-based, energy-efficient processors
- **Lambda / Fargate**: Zero compute when idle
- **S3 Intelligent-Tiering**: Auto-optimizes storage
- **Auto Scaling**: Match demand, eliminate waste
- **Customer Carbon Footprint Tool**: Track emissions
- **Compute Optimizer**: Find over-provisioned resources
