# 🎬 How Netflix Handles Streaming (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Netflix's streaming architecture demonstrates **CDN at global scale** and is a perfect case study for content delivery system design.

---

## 2. Netflix's Scale

```
The challenge:
├── 200+ million subscribers
├── 100+ million hours watched daily
├── Peak: 20%+ of global internet traffic
├── 190+ countries
└── 4K HDR = 25 Mbps per stream

Key constraint: Cannot rely on public internet
```

---

## 3. Two-Cloud Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTROL PLANE (AWS)                       │
│                                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐│
│  │ User    │  │ Catalog │  │ Billing │  │ Recommendation │ ││
│  │ Service │  │ Service │  │ Service │  │    Engine       ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────────────┘│
│                                                              │
│               Everything EXCEPT video bytes                  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                 DATA PLANE (Open Connect CDN)               │
│                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐               │
│  │   OCA     │  │   OCA     │  │   OCA     │               │
│  │(US-East)  │  │(EU-West)  │  │(Tokyo)    │               │
│  └───────────┘  └───────────┘  └───────────┘               │
│                                                              │
│               18,000+ OCAs in 175 countries                 │
│               100% of video traffic                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Open Connect Architecture

### Open Connect Appliances (OCAs)

```
Custom servers deployed inside ISPs:
├── Flash storage: ~100-200TB per OCA
├── 100 Gbps network throughput
├── Strategically placed in ISP networks
├── FREE to ISPs (Netflix pays for hardware)
└── Result: Netflix traffic stays local

ISP benefits:
├── Reduced upstream bandwidth costs
├── Better user experience
└── Less network congestion
```

### Content Flow

```
Content Creation → Encoding → Distribution → Playback

┌─────────────┐
│ Master File │
│ (4K, 100GB) │
└──────┬──────┘
       │ Encode (AWS)
       ▼
┌─────────────┐
│ 50 Versions │
│ (codec,     │
│ bitrate,    │
│ resolution) │
└──────┬──────┘
       │ Distribute (Off-peak)
       ▼
┌─────────────────────────────────────────────┐
│              Open Connect OCAs              │
│  (content pushed during 2 AM - 2 PM local)  │
└─────────────────────────────────────────────┘
```

---

## 5. Adaptive Streaming

```
Playback adapts to network conditions:

Good connection:    4K HDR @ 25 Mbps
Moderate:           1080p @ 5 Mbps
Poor:               480p @ 1.5 Mbps

Client algorithm:
1. Monitor buffer fill rate
2. Estimate available bandwidth
3. Request appropriate quality chunk
4. Seamlessly switch mid-stream
```

### Chunk-Based Delivery

```
Video broken into chunks:
├── Each chunk: 2-4 seconds
├── Pre-encoded at multiple bitrates
├── Client fetches chunk by chunk
└── Can switch quality per chunk

Example request:
GET /video/123/chunk-42-1080p.mp4
```

---

## 6. Resilience Patterns

### Chaos Engineering (Chaos Monkey)

```
Netflix invented chaos engineering:
├── Randomly kill instances in production
├── Test DR during business hours
├── Build confidence in resilience
└── "If you want to find bugs, break things"

Simian Army:
├── Chaos Monkey: Kill instances
├── Chaos Kong: Kill entire regions
├── Latency Monkey: Inject delays
└── Conformity Monkey: Find non-conforming instances
```

### Hystrix (Circuit Breaker)

```
Every service call wrapped with:
├── Timeout (prevent indefinite waits)
├── Circuit breaker (stop calling failing service)
├── Fallback (degrade gracefully)
├── Bulkhead (isolate thread pools)
```

---

## 7. Trade-offs Netflix Makes

| Decision | Netflix's Choice | Trade-off |
|----------|-----------------|-----------|
| **CDN** | Own infrastructure | Massive CAPEX, but control |
| **Encoding** | Pre-encode 50 versions | Storage cost, but fast playback |
| **Reliability** | Break in production | Risk, but builds resilience |
| **Control plane** | All on AWS | Single cloud dependency |

---

## 8. Interview Application

> "For a streaming platform at scale, I'd follow Netflix's two-cloud pattern. The control plane (user service, catalog, recommendations) runs on a public cloud like AWS for flexibility.
>
> For video delivery, I'd build or partner for a CDN with edge caches close to users. The key insight is that cached video content should be served from inside ISP networks, not across the public internet.
>
> Content is pre-encoded into multiple bitrates and resolutions. The player uses adaptive streaming: monitoring available bandwidth and switching quality seamlessly.
>
> For resilience, I'd implement circuit breakers on all service calls and practice chaos engineering—randomly killing instances to find weaknesses before they cause outages.
>
> The trade-off is infrastructure investment. Building owned CDN is expensive but provides control over quality at scale."

---

## 9. Senior-Level Phrases

- "I'd separate control plane (AWS) from data plane (CDN) for independent scaling."
- "Edge caches inside ISP networks eliminate public internet bottlenecks."
- "Adaptive bitrate streaming adjusts quality per chunk based on bandwidth."
- "Chaos engineering builds confidence in resilience—break things intentionally."
