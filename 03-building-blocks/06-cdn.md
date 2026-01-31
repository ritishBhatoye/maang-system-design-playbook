# CDN (Content Delivery Network)

## 1. Problem Statement

How do we serve static content (images, videos, HTML, CSS, JS) to users globally with low latency and high availability?

---

## 2. Clarifying Questions

* What content types? (Static vs dynamic)
* What's the geographic distribution? (Global vs regional)
* What's the cache TTL? (How long to cache)
* What's the update frequency? (How often content changes)

---

## 3. Requirements

### Functional

* Serve static content globally
* Cache content at edge locations
* Handle cache invalidation
* Support high traffic

### Non-Functional

* Low latency (<50ms from edge)
* High availability (99.99%)
* Cost-effective bandwidth
* Automatic scaling

---

## 4. Scale Assumptions

* 1B requests/day
* 100TB content
* Global distribution (5+ regions)
* 95% cache hit rate

---

## 5. High-Level Architecture

**CDN Flow**
```
User (Asia) → CDN Edge (Asia) → Cache Hit: Return
                                Cache Miss: Origin Server (US) → Cache → Return
```

**Origin Server**
```
CDN → Origin (S3/Storage) → Content
```

---

## 6. Core Components

* **Edge Servers**: Cache content close to users
* **Origin Server**: Source of truth (S3, storage)
* **Cache Key**: URL + headers determine cache
* **TTL**: How long to cache
* **Invalidation**: How to purge cache

---

## 7. Data Flow

**Cache Hit Flow**
1. User → Request content (image.jpg)
2. DNS → Route to nearest CDN edge
3. Edge → Check cache
4. Edge → Cache hit, return content
5. User → Receives content (<50ms)

**Cache Miss Flow**
1. User → Request content
2. Edge → Check cache, miss
3. Edge → Request from origin server
4. Origin → Return content
5. Edge → Cache content (with TTL)
6. Edge → Return to user
7. Future requests → Cache hit

**Cache Invalidation Flow**
1. Content updated at origin
2. Invalidation API → Purge cache at all edges
3. Next request → Cache miss, fetch from origin
4. Edge → Cache new content

---

## 8. Bottlenecks

* **Origin Server Overload**: Too many cache misses
  * Solution: Increase TTL, pre-warm cache, or use multiple origins
* **Cache Invalidation**: Slow to propagate
  * Solution: Use versioned URLs (image_v2.jpg) instead of invalidation
* **Hot Content**: Single edge gets all traffic
  * Solution: CDN automatically replicates to multiple edges
* **Dynamic Content**: Can't cache effectively
  * Solution: Cache static parts, use edge computing for dynamic
* **Cost**: Bandwidth costs at scale
  * Solution: Optimize content size, use compression

---

## 9. Trade-offs

| Strategy | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **Long TTL** | High hit rate, low origin load | Stale content | Static assets (images, CSS) |
| **Short TTL** | Fresh content | More origin requests | Frequently changing content |
| **Versioned URLs** | Instant updates, no invalidation | URL management | Best practice for static assets |
| **Cache Everything** | Simple | Wastes cache on unique content | High-traffic, reusable content |
| **Selective Caching** | Efficient | Complex rules | Mixed content types |

**CDN vs Origin:**
- **CDN**: Low latency, high bandwidth, but cache may be stale
- **Origin**: Always fresh, but higher latency, more expensive bandwidth

---

## 10. How to Explain This in an Interview

> "I use a CDN (like CloudFront) to serve static assets globally. I set TTL to 1 year for immutable assets like images and use versioned URLs (image_v2.jpg) for updates, avoiding cache invalidation. For user-generated content, I use shorter TTL (1 hour) or cache at edge with origin fallback. This reduces origin load by 95% and improves global latency from 200ms to <50ms."

**Key Points:**
* Always mention geographic benefit (latency reduction)
* Explain TTL strategy based on content type
* Mention versioned URLs for cache busting
* Quantify the benefit (hit rate, latency improvement)

---

## 11. Common Mistakes

* **Caching dynamic content**: "We cache user profiles"
  * Fix: Only cache static or semi-static content
* **No cache invalidation strategy**: "Content never updates"
  * Fix: Use versioned URLs or invalidation API
* **Too short TTL**: "TTL is 1 minute for images"
  * Fix: Use long TTL for static assets, version URLs for updates
* **Ignoring origin load**: "CDN handles everything"
  * Fix: Monitor cache hit rate, optimize TTL
* **Not using CDN**: "We serve from single region"
  * Fix: Use CDN for global traffic to reduce latency
* **Cache key issues**: "Same URL, different content"
  * Fix: Include version or hash in URL or cache key
