# 🚗 How Uber Handles Real-Time (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Uber's architecture demonstrates **real-time geospatial systems** at massive scale. Perfect for testing location-based service design.

---

## 2. Uber's Scale

```
The challenge:
├── Millions of active drivers
├── Millions of concurrent ride requests
├── GPS updates every 4 seconds per driver
├── Match rider to driver in seconds
├── Accurate ETAs in real-time
└── Global coverage (10,000+ cities)
```

---

## 3. Key Components

### Location Service

```
Manages real-time driver positions:
├── Ingests GPS every 4 seconds
├── Stores in in-memory cache (Redis with geospatial)
├── Indexes by H3 hexagonal cells
└── Query: "Find drivers near (lat, lng)"

GPS Update Flow:
Driver app → API Gateway → Location Service → Redis Geo
```

### Dispatch Service

```
Matches riders with drivers:
├── Rider requests → Find nearby drivers
├── Score candidates (ETA, rating, type)
├── Send request to best match
├── Handle accept/decline
└── Fallback to next driver if no response
```

### H3 Geospatial Index

```
Uber's hexagonal grid system:
├── Earth divided into hexagonal cells
├── Hierarchical (zoom levels)
├── Same-sized cells (vs distorted squares)
├── Efficient neighbor queries

Matching uses H3:
1. Rider at (lat, lng)
2. Find H3 cell at resolution 9 (~174m)
3. Query drivers in that cell + neighbors
4. Expand search if needed
```

---

## 4. Architecture Diagram

```
                    ┌─────────────────────────────────────┐
                    │        Driver Mobile App            │
                    │      (GPS every 4 seconds)          │
                    └───────────────┬─────────────────────┘
                                    │
                    ┌───────────────┴─────────────────────┐
                    │           API Gateway               │
                    └───────────────┬─────────────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
   ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
   │  Location    │          │   Dispatch   │          │    Trip      │
   │  Service     │←─────────│   Service    │          │   Service    │
   └──────┬───────┘          └──────────────┘          └──────────────┘
          │
          ▼
   ┌──────────────┐
   │  Redis Geo   │
   │ (H3 indexed) │
   └──────────────┘
```

---

## 5. Matching Algorithm

### Evolution of Matching

```
V1 - Greedy (Simple):
├── Find closest driver
├── Assign immediately
├── Problem: Globally suboptimal

V2 - Batch Matching:
├── Collect requests for window (2-5 sec)
├── Optimize globally for region
├── Hungarian algorithm for assignment
├── Result: Lower average wait times
```

### Matching Factors

```
Score = f(ETA, driver_rating, vehicle_type, acceptance_rate, surge)

Ranking considers:
├── ETA (primary)
├── Traffic conditions
├── Driver acceptance history
├── Vehicle match (UberX, Black, XL)
├── Surge pricing zone
└── Driver's next trip direction
```

---

## 6. Real-Time ETA

```
Challenge: Accurate ETA with traffic

Sources:
├── Historical patterns (time of day, day of week)
├── Real-time driver speed data
├── Traffic incidents/events
├── GPS trajectory analysis

Process:
1. Map-match GPS to road segments
2. Calculate speed per segment from recent trips
3. Route with weighted segment times
4. Add pickup time estimate

Accuracy: Within 15% of actual
```

---

## 7. Handling Scale

### City Isolation

```
Architectural principle:
├── Cities are independent failure domains
├── NYC outage doesn't affect LA
├── Data partitioned by region
└── Regional clusters handle local traffic

Ringpop (consistent hash ring):
├── Partition cities into regions
├── Route requests to owning shard
├── Automatic rebalancing on node failure
```

### Surge Pricing

```
Real-time supply/demand:
├── Count drivers per H3 cell
├── Count riders per H3 cell
├── Demand > supply → Surge multiplier
├── Update every minute
└── Display to riders before request
```

---

## 8. Trade-offs Uber Makes

| Decision | Uber's Choice | Trade-off |
|----------|---------------|-----------|
| **Matching** | Batch (delayed) | 2-sec latency for better matches |
| **Location storage** | In-memory | Cost, but sub-ms queries |
| **GPS frequency** | 4 seconds | Battery life vs accuracy |
| **Consistency** | Local only | Simple, but no cross-region |

---

## 9. Interview Application

> "For a ride-sharing system, I'd start with the core loop: driver location updates every 4 seconds, stored in Redis with geospatial indexing.
>
> For matching, I'd use Uber's H3 hexagonal grid to efficiently find nearby drivers. Query expands from the rider's cell to neighbors until enough candidates found.
>
> Rather than greedy 'closest driver' matching, I'd use batch matching: collect requests for 2 seconds, then optimally assign the batch. This reduces average wait times at the cost of slight delay.
>
> For ETA, combine historical patterns with real-time traffic from current trips. Map-match GPS to road segments and weight by current speeds.
>
> The system is partitioned by city/region for isolation—an outage in one city doesn't affect others."

---

## 10. Senior-Level Phrases

- "H3 indexing provides efficient k-nearest queries for geospatial matching."
- "Batch matching optimizes globally rather than greedy local assignment."
- "4-second GPS intervals balance accuracy with driver battery life."
- "City isolation ensures regional failures don't cascade globally."
