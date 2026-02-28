# 🚕 Uber / Lyft: Ride Matching (Real-time GIS)

> **Staff-Signal**: Can you coordinate thousands of moving objects in real-time without causing a race condition or stale location reads?

---

## 1. Problem Statement
Design a ride-sharing service that matches a Rider with the nearest available Driver.

---

## 2. Clarifying Questions
*   **Scale**: How many users/drivers? (100M riders, 1M drivers).
*   **Real-time requirements**: How often do drivers report location? (Every 3-5 seconds).
*   **Matching Criteria**: Just distance? (Primarily distance, but also rating and car type).

---

## 3. Requirements
### Functional
*   Drivers share their current location.
*   Riders see nearby drivers in real-time.
*   Riders request a ride; system matches them with the "Best" Driver.

### Non-Functional
*   **Low Latency**: Location updates must be visible in < 1s.
*   **High Availability**: A regional failure shouldn't stop the global service.
*   **Consistency**: A driver should not be matched to two riders (Atomic matching).

---

## 4. Capacity Estimation (Worked Math)
*   **Drivers**: 1M.
*   **Location Updates**: 1M drivers * update every 4s = **250,000 writes/sec** (Write-Heavy).
*   **Rider Queries**: 100M riders. Assume 1M searching at once * query every 5s = **200,000 reads/sec**.
*   **Storage**: Location is small (lat, lon, driver_id, timestamp) ~ 64 bytes. 1M drivers * 64B = 64MB total (Fits in RAM!).

---

## 5. API Design
### `POST /v1/driver/location`
Updates driver coordinates.
```json
{
  "driver_id": "uuid",
  "lat": 37.7749,
  "lon": -122.4194
}
```

### `GET /v1/rider/nearby?lat=...&lon=...`
Returns list of available drivers.
```json
{
  "drivers": [
    {"id": "d1", "lat": 37.77, "lon": -122.42},
    {"id": "d2", "lat": 37.78, "lon": -122.41}
  ]
}
```

---

## 6. Data Model (Schema Design)
We use a NoSQL Key-Value store for current locations (Redis for speed).
*   **Table: DriverStatus**
    *   `driver_id` (Partition Key)
    *   `lat`, `lon`
    *   `last_updated`
    *   `is_available` (Boolean)

---

## 7. High-Level Architecture
```
Rider App / Driver App → Load Balancer → API Gateway
                                          │
                  ┌───────────────────────┼───────────────────────┐
                  ↓                       ↓                       ↓
           Location Service       Matching Service        Payment Service
                  │                       │ (Distributed Lock)
                  ↓                       ↓
           Geo-Spatial Index      Ride Metadata (Postgres)
             (Redis / S2)
```

---

## 8. Component Deep Dive: Geo-Spatial Indexing
Standard DBs aren't built for "Find all points within X radius."
*   **Option A: Quadtrees**. Hierarchical tree divided into 4 quadrants. Fast, but hard to update when drivers move.
*   **Option B: Google S2 / H3 (Hexagons)**. Maps the 2D earth to 1D Hilbert Curves. Every lat/lon is converted to a 64-bit Cell ID.
    *   **Staff Decision**: Use **Google S2** and store driver location in a **Redis Sorted Set**. The `Score` is the S2 Cell ID. This allows for range queries (finding nearby drivers) in **O(log N)** time.

---

## 9. Data Flow
1.  **Driver** sends location → API Gateway → Location Service.
2.  **Location Service** updates the S2 Cell ID in Redis.
3.  **Rider** opens app → Request nearby drivers.
4.  **Backend** calculates the S2 Cells covering the Rider's 5km radius and queries Redis.

---

## 10. Bottlenecks
*   **Redis Hotspot**: If all drivers are in NYC, one Redis shard is crushed.
    *   **Solution**: Shard by S2 Cell ID, not Driver ID.
*   **Atomic Match**: Two riders click "Request" at the exact same millisecond for the same driver.
    *   **Solution**: Use a **Conditional Update** (Optimistic Locking) on the Driver's `is_available` flag.

---

## 11. Scaling Strategy
*   **Microservices**: Decouple matching from location updates.
*   **Shard by Geography**: Regions (NYC, SF, London) can run on independent database clusters to reduce cross-continental latency.

---

## 12. Failure Scenarios
*   **Redis Node Dies**: Use Redis Sentinel/Cluster for auto-failover. Since location data is ephemeral, we can tolerate losing 5 seconds of updates as drivers will report again soon.

---

## 13. Tradeoffs

| Choice | Pro | Con |
| :--- | :--- | :--- |
| **S2 over Quadtree** | Easy to shard, extremely fast range queries | Requires math library overhead |
| **Sync over Async Matching** | Guarantees "One Rider - One Driver" | Slower response for the rider |

---

## 14. Monitoring Strategy
*   **Matching Latency**: Alert if ride match takes > 5s.
*   **Driver Stale Rate**: Monitor percentage of drivers whose location hasn't updated in > 30s.

---

## 15. The Interview Narrative (Staff Level)
> "In my design, I tackle the high-write volume of driver updates—250k per second—by utilizing a memory-first Geo-Spatial index via Google S2. This allows us to convert 2D coordinates into 1D sortable IDs, making the 'Find Nearby' query a simple range scan in Redis. To handle the race condition of concurrent ride requests, I implement a distributed lock or conditional update on the driver's availability state, ensuring consistency even at peak global scale."

---

## 16. Follow-up Questions
1.  **"How do you handle 'Gaps' in the highway?"** (Answer: Predictive routing/Kalman filters).
2.  **"What if the driver's phone loses signal?"** (Answer: Heartbeats and automatic expiry in Redis).

---

## 17. Common Mistakes
1.  **Using a standard relational DB** for `SELECT * WHERE lat < X...` at 100k QPS. (Too slow).
2.  **Forgetting about 'Surge Pricing'** and the metadata storage needed for it.
3.  **Not handling the 'Matched but Offline'** scenario.
