---
name: waf-performance
description: AWS Well-Architected Performance Efficiency pillar review criteria. Use when evaluating architecture for right-sizing, caching, database selection, async processing, and CDN.
---

# Performance Efficiency — Review Criteria

## Checklist

### Compute Right-Sizing
- [ ] Instance types match workload profile (compute/memory/storage optimized)
- [ ] Graviton instances used where compatible (20% better price-performance)
- [ ] Serverless considered for variable/bursty workloads (Lambda, Fargate)
- [ ] Spot instances for fault-tolerant batch workloads
- [ ] Compute Optimizer recommendations reviewed

### Caching
- [ ] Application-level cache (ElastiCache Redis/Memcached)
- [ ] Database query cache or read replicas for read-heavy loads
- [ ] API response caching (API Gateway cache, CloudFront)
- [ ] Session store externalized to cache (not local disk)
- [ ] Cache invalidation strategy defined (TTL, event-based)

### Database Selection
- [ ] Purpose-built database for workload pattern:
  - Relational (RDS/Aurora) for ACID transactions
  - Key-value (DynamoDB) for high-throughput, low-latency
  - Document (DocumentDB) for flexible schema
  - Graph (Neptune) for relationship queries
  - Time-series (Timestream) for IoT/metrics
- [ ] Read replicas for read-heavy workloads
- [ ] Connection pooling (RDS Proxy) for serverless/many connections
- [ ] Query optimization and indexing reviewed

### Async & Event-Driven
- [ ] Long-running tasks offloaded to queues (SQS, Step Functions)
- [ ] Event-driven processing via EventBridge
- [ ] Webhooks/callbacks instead of polling
- [ ] Batch processing for non-time-sensitive operations

### Content Delivery & Edge
- [ ] CloudFront for static assets and API acceleration
- [ ] S3 Transfer Acceleration for large uploads
- [ ] Lambda@Edge or CloudFront Functions for edge logic
- [ ] Regional edge caches for dynamic content

### Monitoring & Benchmarking
- [ ] Load testing performed (baseline established)
- [ ] P99 latency tracked (not just averages)
- [ ] Auto-scaling triggers based on actual metrics
- [ ] Performance regression detection in CI/CD

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Over-provisioned EC2 (r5.4xlarge running at 5% CPU) | Wasted spend, false capacity confidence | Right-size with Compute Optimizer |
| No caching — every request hits DB | High latency under load, DB bottleneck | ElastiCache + read replicas |
| Synchronous API calls to slow dependencies | Cascading latency | SQS + async processing |
| Serving static files from application server | Wasted compute, high latency | S3 + CloudFront |
| Single large relational DB for everything | Contention, scaling ceiling | Purpose-built databases per workload |
| No CDN for global users | High latency for distant users | CloudFront with regional edge caches |

## Key AWS Services

- **CloudFront**: CDN, edge caching, TLS termination
- **ElastiCache**: Redis/Memcached in-memory caching
- **RDS Proxy**: Connection pooling for serverless
- **DynamoDB + DAX**: Microsecond read caching
- **Auto Scaling**: Dynamic compute adjustment
- **Compute Optimizer**: Right-sizing recommendations
- **Lambda**: Event-driven serverless compute
- **Step Functions**: Workflow orchestration
- **Global Accelerator**: Network-layer performance

## Example Findings

**Good**: "CloudFront serves static assets with 95% cache hit ratio, Aurora with 2 read replicas handles read traffic, ElastiCache Redis caches session data and frequent queries"

**Bad**: "All traffic hits a single m5.xlarge EC2 instance running Nginx + App + PostgreSQL on the same host, no caching layer, users in Europe experience 800ms latency to us-east-1"
