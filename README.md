# 🧠 MAANG System Design Playbook (2026 Edition)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Interview Ready](https://img.shields.io/badge/Interview-Ready-blue.svg)]()

> **Written from the perspective of a Senior Staff Engineer who has interviewed 500+ MAANG candidates.**

This repository is a **high‑signal, interview‑first system design playbook** for engineers targeting **MAANG‑level roles in 2026**.

This is **not** a textbook.
This is **how to THINK, DECIDE, and COMMUNICATE** in real interviews.

---

## 🎯 Who This Is For

- Engineers with 2–10+ years of experience
- Preparing for **MAANG / Big Tech / Unicorn** system design rounds
- Tired of vague, academic explanations

---

## 🧠 How MAANG Interviews Actually Work (2026)

Interviewers do **not** care if you remember every component.
They care if you can:

- Ask the _right clarifying questions_
- Make _explicit trade‑offs_
- Justify decisions under constraints
- Communicate like a senior engineer

This repo trains exactly that.

---

## 📁 Repository Structure

```
maang-system-design-playbook/
│
├── README.md
│
├── 01-interview-skills/              # 🎯 Start here - Learn how to interview
│   ├── 01-maang-interview-flow.md
│   ├── 02-clarifying-questions.md
│   ├── 03-how-to-communicate.md
│   ├── 04-how-to-explain-tradeoffs.md
│   ├── 05-common-interview-phrases.md
│   └── 06-sample-interviews.md
│
├── 02-fundamentals/                   # 📚 Core principles
│   ├── 01-architecture-basics.md
│   ├── 02-scalability-basics.md
│   ├── 03-latency-throughput.md
│   ├── 04-consistency-patterns.md
│   ├── 05-performance-metrics.md
│   ├── 06-capacity-estimation.md
│   └── 07-concurrency.md
│
├── 03-building-blocks/                # 🧱 System components
│   ├── 01-load-balancers.md
│   ├── 02-databases-sql-nosql.md
│   ├── 03-caches.md
│   ├── 04-message-queues.md
│   ├── 05-queues-streams.md
│   ├── 06-cdn.md
│   ├── 07-indexing.md
│   └── 08-search-services.md
│
├── 04-patterns-and-paradigms/         # 🏗️ Architectural patterns
│   ├── 01-microservices-vs-monolith.md
│   ├── 02-event-driven.md
│   ├── 03-pub-sub.md
│   ├── 04-grpc-vs-rest.md
│   └── 05-sagas-cqrs.md
│
├── 05-tradeoffs/                      # ⚖️ Decision-making frameworks
│   ├── 01-sql-vs-nosql.md
│   ├── 02-consistency-vs-availability.md
│   ├── 03-sync-vs-async.md
│   ├── 04-push-vs-pull.md
│   ├── 05-polling-vs-streaming.md
│   └── 06-cost-vs-performance.md
│
├── 06-case-studies/                   # 💼 Practice problems
│   ├── 01-url-shortener.md
│   ├── 02-chat-system.md
│   ├── 03-rate-limiter.md
│   ├── 04-notification-system.md
│   ├── 05-file-storage.md
│   ├── 06-news-feed.md
│   ├── 07-search-autocomplete.md
│   └── 08-load-balancing-at-scale.md
│
├── 07-diagrams-as-text/               # 📊 Visual reference
│   ├── 01-basic-components.txt
│   ├── 02-system-patterns.txt
│   └── 03-common-architecture-patterns.md
│
├── 08-advanced-topics/                # 🚀 Advanced distributed systems
│   ├── 01-multi-region-design.md
│   ├── 02-geo-replication.md
│   ├── 03-disaster-recovery.md
│   ├── 04-hot-vs-cold-storage.md
│   ├── 05-data-migration-at-scale.md
│   └── 06-backpressure-and-flow-control.md
│
├── 09-real-world-failures/            # 💥 Common failure modes
│   ├── 01-cache-stampede.md
│   ├── 02-thundering-herd.md
│   ├── 03-cascading-failures.md
│   ├── 04-retry-storms.md
│   └── 05-partial-outages.md
│
├── 10-scaling-playbooks/              # 📈 Scaling by workload type
│   ├── 01-scaling-read-heavy-systems.md
│   ├── 02-scaling-write-heavy-systems.md
│   ├── 03-scaling-realtime-systems.md
│   ├── 04-scaling-analytics-systems.md
│   └── 05-scaling-ai-systems.md
│
├── 11-company-patterns/               # 🏢 How MAANG does it
│   ├── 01-how-google-designs-systems.md
│   ├── 02-how-meta-handles-feed.md
│   ├── 03-how-amazon-scales-storage.md
│   ├── 04-how-netflix-handles-streaming.md
│   └── 05-how-uber-handles-realtime.md
│
└── 12-ai-meets-system-design/         # 🤖 AI/LLM system design (NEW 2026)
    ├── 01-designing-llm-backed-systems.md
    ├── 02-vector-databases.md
    ├── 03-prompt-caching.md
    ├── 04-rate-limiting-for-ai.md
    └── 05-cost-control-for-ai-systems.md
```

> **Note**: Both directories and files are numbered in recommended learning order. Start with `01-` and progress sequentially through each section.

---

## 🧩 Mandatory Case Study Format (Used Everywhere)

Every system design topic follows **this exact interview‑proven structure**:

1. Problem Statement (interviewer style)
2. Clarifying Questions
3. Requirements
   - Functional
   - Non‑Functional

4. Scale Assumptions
5. High‑Level Architecture
6. Core Components
7. Data Flow
8. Bottlenecks
9. Trade‑offs
10. How to Explain This in an Interview
11. Common Mistakes

---

## 🧪 Sample Case Study

# 🔗 URL Shortener (2026 MAANG Edition)

## 1. Problem Statement

Design a system like **bit.ly** that shortens long URLs and redirects users at scale.

---

## 2. Clarifying Questions (Ask First)

- Expected read vs write ratio?
- Custom aliases required?
- URL expiration needed?
- Global or regional traffic?

---

## 3. Requirements

### Functional

- Generate short URLs
- Redirect short URL → original URL
- Handle collisions

### Non‑Functional

- Low latency redirects (<50ms)
- Highly available
- Horizontally scalable

---

## 4. Scale Assumptions

- 100M URLs/day
- 10:1 read/write ratio
- Peak QPS: ~15K writes, ~150K reads

---

## 5. High‑Level Architecture

```
Client → Load Balancer → API Service
                       → Cache (Redis)
                       → Database
```

---

## 6. Core Components

- API Service
- Hash / ID Generator
- Cache (Redis)
- Persistent Store (NoSQL)

---

## 7. Data Flow

**Write Flow**

1. Client sends long URL
2. Generate unique ID
3. Store mapping
4. Return short URL

**Read Flow**

1. Client hits short URL
2. Check cache
3. Fallback to DB
4. Redirect

---

## 8. Bottlenecks

- ID generation collisions
- Cache misses
- Hot keys

---

## 9. Trade‑offs

- Base62 vs Hashing
- SQL vs NoSQL
- Cache eviction strategy

---

## 10. How to Explain This in an Interview

> "I optimize for read latency using cache because traffic is read‑heavy. I accept eventual consistency for better scalability."

This sentence matters more than the diagram.

---

## 11. Common Mistakes

- Skipping scale assumptions
- Not justifying database choice
- Forgetting cache invalidation

---

## 🚀 How to Use This Repo

**For Interview Prep (Recommended Path):**

1. **Start Here**: Read `01-interview-skills/01-maang-interview-flow.md` to understand the 45-minute structure
2. **Learn the Format**: Memorize the 11-step framework (Problem → Clarifying → Requirements → ... → Common Mistakes)
3. **Master Fundamentals**: Read `02-fundamentals/` in order (01 → 07) to build your foundation
4. **Study Building Blocks**: Understand `03-building-blocks/` in order (01 → 08) - components you'll use
5. **Learn Patterns**: Study `04-patterns-and-paradigms/` (01 → 05) - architectural approaches
6. **Internalize Trade-offs**: Study `05-tradeoffs/` (01 → 06) - decision-making frameworks
7. **Practice Case Studies**: Work through `06-case-studies/` (01 → 08) one by one, speaking your answers aloud
8. **Simulate Interviews**: Pick a random case study, set a 45-minute timer, and design it end-to-end
9. **Reference Diagrams**: Use `07-diagrams-as-text/` as visual reference during practice

**For Senior/Staff+ Candidates (Advanced Path):**

10. **Advanced Topics**: Deep dive into `08-advanced-topics/` - multi-region, DR, geo-replication
11. **Learn from Failures**: Study `09-real-world-failures/` - cache stampede, thundering herd, cascading failures
12. **Scaling Playbooks**: Master `10-scaling-playbooks/` - read-heavy, write-heavy, real-time, analytics, AI
13. **Company Patterns**: Learn `11-company-patterns/` - how Google, Meta, Amazon, Netflix, Uber build systems
14. **AI System Design** (🔥 NEW 2026): Study `12-ai-meets-system-design/` - LLM systems, vector DBs, prompt caching

**Pro Tips:**

- Practice **speaking answers aloud** — writing is not enough
- Focus on **trade‑offs, not just components** — this is what separates senior engineers
- Use the **numbered 1-11 format** for every system design problem
- Time yourself — 45 minutes goes fast in real interviews
- **Print out [QUICK-REFERENCE.md](QUICK-REFERENCE.md)** and keep it next to you during practice
- For 2026: **AI/LLM system design is increasingly common** — don't skip section 12

---

## 🏁 Final Note

This playbook is optimized for **MAANG interviews in 2026** — where clarity, decision‑making, and communication matter more than buzzwords.

If this helps you crack your dream company, ⭐ the repo and pass it forward.

Good luck — think like a Staff Engineer.

---

## 📚 Additional Resources

**Quick Links:**

- [QUICK-REFERENCE.md](QUICK-REFERENCE.md) — Print this for practice sessions
- [PRACTICE-CHECKLIST.md](PRACTICE-CHECKLIST.md) — Track your progress
- [CONTRIBUTING.md](CONTRIBUTING.md) — Help improve this playbook
- [LICENSE](LICENSE) — MIT License

**Recommended Reading:**

- Designing Data-Intensive Applications by Martin Kleppmann
- System Design Interview by Alex Xu (Volumes 1 & 2)
- AWS Well-Architected Framework

**Practice Platforms:**

- [Pramp](https://www.pramp.com/) — Free mock interviews
- [Interviewing.io](https://interviewing.io/) — Anonymous practice
- [Exponent](https://www.tryexponent.com/) — System design courses

---

## 🤝 Contributing

Found a mistake? Want to add a case study? See [CONTRIBUTING.md](CONTRIBUTING.md).

This is a community-driven project. PRs are welcome!

---

## ⭐ Star History

If this repo helped you land your dream job, consider:

1. ⭐ Starring the repo
2. 🔄 Sharing it with friends preparing for interviews
3. 💬 Opening an issue to share your success story

---

## 📝 License

MIT License - see [LICENSE](LICENSE) for details.

---

**Built with ❤️ by engineers, for engineers preparing for MAANG interviews.**
