# 🌐 Networking Deep-Dive for System Design

> **Staff-Signal**: Can you explain why your multi-region design is failing even though the 'boxes are connected'?

---

## 1. The OSI Model (Practical View)
In interviews, you only care about:
*   **Layer 4 (Transport)**: TCP/UDP. Reliability vs. Speed.
*   **Layer 7 (Application)**: HTTP, gRPC, WebSocket. Request/Response logic.

---

## 2. TCP vs UDP (The Decision)
| Feature | TCP | UDP |
| :--- | :--- | :--- |
| **Connection** | Handshake (SYN-ACK) | Connectionless |
| **Reliability** | Retransmission, Sequencing | Best effort |
| **Congestion** | Flow Control | None |
| **Use Case** | DBs, Payments, APIs | Video Streaming, Gaming, VoIP |

---

## 3. DNS (Domain Name System)
DNS is the first "Load Balancer" in any global system.
- **Anycast DNS**: Routes the user to the nearest server based on BGP.
- **Latency-based Routing**: The DNS server returns the IP of the datacenter with the lowest RTT.
- **TTL (Time to Live)**: High TTL = better cache, slow recovery from outage. Low TTL = fast recovery, higher load on DNS.

---

## 4. HTTP/1.1 vs HTTP/2 vs HTTP/3
This is a common "Staff Probing" question.
*   **HTTP/1.1**: One request per TCP connection (Head-of-Line blocking).
*   **HTTP/2**: Multiplexing over one TCP connection. Still suffers from **TCP Head-of-Line blocking** if one packet is lost.
*   **HTTP/3**: Uses **QUIC (UDP)**. Handshake is combined with TLS (faster). No Head-of-Line blocking across streams.

---

## 5. BGP (Border Gateway Protocol)
The "Glue" of the internet.
- **Interview Application**: How does AWS/Google move traffic between regions so fast? Their private global backbone using **BGP Anycast**.
- **Outage Scenario**: "Google/Facebook went down because of a BGP misconfiguration" is a classic post-mortem topic.

---

## 6. TLS Handshake (Performance Cost)
- **Problem**: Every HTTPS connection requires multiple RTTs (Round Trip Times) before a single byte of data is sent.
- **Optimization**: Use **TLS 1.3** or **TLS False Start** to reduce RTTs. This is why CDNs are essential (terminating TLS closer to the user).

---

## ⚖️ Tradeoffs: Load Balancing (L4 vs L7)
| LB Type | Layer | Intelligence | Performance |
| :--- | :--- | :--- | :--- |
| **L4** | Transport | IP/Port based. | Extremely fast (Hardware). |
| **L7** | Application | Path/Header based (Cookie, URL). | High (Can do WAF, Auth). |

---

## 💬 How to use this in an Interview

> "To optimize our global 4K streaming latency, I will terminate the **TLS 1.3** connection at the **Edge PoP** using **HTTP/3 over QUIC**. This avoids the TCP Head-of-Line blocking bottleneck and significantly reduces the RTTs required for the initial handshake. For global discovery, I'll utilize **BGP Anycast** to route users to the nearest healthy datacenter via our private fiber backbone."

### Follow-up question: "What happens to a TCP connection during a data-center failover?"
**Correct Answer**: "Established TCP connections will be interrupted. Unlike UDP, TCP is stateful and bound to the server's IP. The client must re-initiate the handshake with the new IP provided by the updated DNS record or Anycast reroute."
