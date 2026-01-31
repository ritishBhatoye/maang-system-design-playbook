# Cost vs Performance

## 1. Problem Statement

How do we balance system performance with infrastructure costs, and make cost-conscious architectural decisions?

---

## 2. Cost Drivers

### Compute
* **Servers**: CPU, memory, instances
* **Scaling**: More servers = more cost
* **Optimization**: Right-size instances, use spot instances

### Storage
* **Database**: Storage size, IOPS, backups
* **Object Storage**: Data stored, data transferred
* **Optimization**: Compression, archiving, lifecycle policies

### Network
* **Bandwidth**: Data transfer in/out
* **CDN**: Edge data transfer
* **Optimization**: CDN for static content, compression

### Services
* **Managed Services**: Databases, queues, search
* **Third-Party APIs**: External service costs
* **Optimization**: Use managed services wisely

---

## 3. Performance vs Cost Trade-offs

### Over-Provisioning
* **Approach**: Provision more than needed
* **Pros**: Handles spikes, simple
* **Cons**: Wastes money, inefficient
* **When**: Early stage, unknown patterns

### Right-Sizing
* **Approach**: Match resources to actual needs
* **Pros**: Cost-effective, efficient
* **Cons**: Requires monitoring, may need scaling
* **When**: Mature systems, known patterns

### Auto-Scaling
* **Approach**: Scale up/down based on load
* **Pros**: Pay for what you use, handles spikes
* **Cons**: Scaling delay, more complex
* **When**: Variable load, cost optimization needed

---

## 4. Cost Optimization Strategies

### Caching
* **Cost**: Memory (cheaper than compute)
* **Benefit**: Reduces database load, faster responses
* **ROI**: High - small cost, big performance gain
* **Example**: Redis cache reduces DB queries by 80%

### CDN
* **Cost**: Edge data transfer (cheaper than origin)
* **Benefit**: Reduces origin bandwidth, lower latency
* **ROI**: High - reduces bandwidth costs significantly
* **Example**: CDN reduces origin bandwidth by 90%

### Read Replicas
* **Cost**: Additional database instances
* **Benefit**: Scales reads, reduces primary load
* **ROI**: Medium - cost of replica, but enables read scaling
* **Example**: Read replicas handle 10x read traffic

### Compression
* **Cost**: CPU (minimal)
* **Benefit**: Reduces bandwidth, storage
* **ROI**: Very high - minimal cost, significant savings
* **Example**: Gzip reduces payload by 70%

---

## 5. Cost-Performance Matrix

| Strategy | Cost Impact | Performance Impact | ROI | When to Use |
|----------|-------------|-------------------|-----|-------------|
| **Caching** | Low (memory) | High (latency) | Very High | Read-heavy workloads |
| **CDN** | Low (edge transfer) | High (global latency) | Very High | Static/semi-static content |
| **Read Replicas** | Medium (instances) | High (read scaling) | High | Read-heavy, can't scale primary |
| **Sharding** | Medium (more DBs) | High (write scaling) | High | Write-heavy, single DB bottleneck |
| **More Servers** | High (instances) | High (throughput) | Medium | Need more compute |
| **Bigger Servers** | High (instance size) | Medium (vertical scale) | Low | Limited by vertical scaling |

---

## 6. Hidden Costs

### Data Transfer
* **Ingress**: Usually free
* **Egress**: Can be expensive at scale
* **Solution**: CDN, regional deployments

### Storage Growth
* **Unbounded Growth**: Data accumulates
* **Solution**: Lifecycle policies, archiving, deletion

### Idle Resources
* **Unused Instances**: Paying for idle capacity
* **Solution**: Auto-scaling, right-sizing, spot instances

### Operational Overhead
* **Complexity**: More services = more to manage
* **Solution**: Managed services, automation

---

## 7. Cost Modeling

### Capacity Planning
* **Estimate**: Users, QPS, data size
* **Calculate**: Required resources
* **Cost**: Multiply by unit costs
* **Optimize**: Identify cost drivers, optimize

### Example Calculation
* **Users**: 10M
* **QPS**: 100K
* **Storage**: 100TB
* **Cost**: 
  * Compute: 100 servers × $100/month = $10K
  * Storage: 100TB × $0.023/GB = $2.3K
  * Bandwidth: 1TB/day × $0.09/GB = $2.7K
  * **Total**: ~$15K/month

---

## 8. Performance-Cost Optimization

### Identify Bottlenecks
* **Measure**: Latency, throughput, resource usage
* **Profile**: Find expensive operations
* **Optimize**: Fix bottlenecks first

### 80/20 Rule
* **80% of cost**: From 20% of operations
* **Focus**: Optimize expensive operations
* **Example**: Database queries, not logging

### Right Tool for the Job
* **Expensive Operations**: Use optimized tools
* **Example**: Redis for caching, not database
* **Trade-off**: Tool cost vs performance gain

---

## 9. How to Explain This in an Interview

> "I optimize for cost-effectiveness, not just performance. For example, I use caching (Redis) which has a low cost but dramatically reduces database load and improves latency. I use CDN for static content to reduce bandwidth costs and improve global latency. I right-size instances based on actual usage rather than over-provisioning. The trade-off is I may accept slightly higher latency in non-critical paths to reduce costs, but I optimize the critical path for performance."

**Key Points:**
* Balance cost and performance
* Use caching, CDN for high ROI
* Right-size resources
* Optimize critical path, accept trade-offs elsewhere

---

## 10. Common Mistakes

* **Over-provisioning**: "We'll provision for peak always"
  * Fix: Use auto-scaling, right-size based on actual usage
* **Ignoring data transfer costs**: "Bandwidth is free"
  * Fix: Monitor egress costs, use CDN
* **No cost monitoring**: "We don't track costs"
  * Fix: Monitor costs, identify drivers, optimize
* **Premature optimization**: "We optimize everything"
  * Fix: Measure first, optimize bottlenecks
* **Ignoring hidden costs**: "Only instance costs matter"
  * Fix: Consider storage, bandwidth, operational overhead
