# 🧠 MAANG System Design Playbook (2026 Edition)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Interview Ready](https://img.shields.io/badge/Interview-Ready-blue.svg)]()

> **"This is the internal training manual I wish every candidate had before they sat in our interview room."**
> — Senior Staff Engineer (Google/Meta/Netflix)

---

## 🎯 The Philosophy (How to Crack L5/L6)

MAANG interviews in 2026 are **not** about memorizing diagrams. They are about **Decision-Making under Constraints**.

This repo is structured to take you from **Zero** to **Staff-Level Architect**. It's built by engineers who have conducted over 500+ interviews and have designed systems you use every day.

### 📶 The Three Levels of Design
*   **L4 (SDE II)**: Can stay structured and build a working happy path. (Score: `Hire`)
*   **L5 (Senior)**: Can justify every tradeoff with numbers and explain "The Why." (Score: `Strong Hire`)
*   **L6+ (Staff)**: Can design for global failure, cost, and organizational scale. (Score: `Exceptional Hire`)

---

## 🗂️ The Master Structure (16 Units)

Starting from scratch? Follow these in order. Already a Senior? Jump to **Unit 06**.

```text
maang-system-design-playbook/
│
├── 00-Orientation/                   # LEVEL CHECK: Where do you stand?
├── 01-Interview-Skills/              # STRATEGY: Whiteboard, Speech, Signals
│
├── 📚 THE FOUNDATIONS (Must Know)
├── 02-Deep-Fundamentals/             # LSM Trees, Raft, BGP, Clock Skew
├── 03-Building-Blocks/               # Redis, Kafka, Cassandra, LB, S3
├── 04-Patterns-and-Paradigms/        # Sagas, CQRS, Circuit Breakers, Outbox
├── 05-Decision-Tradeoffs/            # SQL vs NoSQL, CAP, Consistency vs Latency
├── 06-API-and-Data-Modeling/         # The Schema Design Masterclass (NEW)
│
├── 📐 THE CASE STUDIES (Practice)
├── 07-Case-Studies/                  # Twitter, YouTube, Uber, Uber, Job Scheduler
├── 08-Diagrams/                      # Mermaid Templates & Visual Strategy
│
├── 🚀 THE STAFF-LEVEL PLAYBOOK (L6+)
├── 09-Advanced-Topics/               # Multi-Region, Disaster Recovery, Chaos Eng
├── 10-Real-World-Failures/           # Cache Stampedes, Thundering Herds
├── 11-Scaling-Playbooks/             # Read-Heavy vs Write-Heavy systems
├── 12-Observability-and-SRE/         # SLOs, SLIs, Alerting for Architects
├── 13-Security-and-Compliance/       # GDPR, OAuth2, Data Residency
│
├── 🏢 THE COMPANY PATTERNS
├── 14-Company-Specific/              # How Google vs. Amazon vs. Meta build
│
├── 🤖 AI & FUTURE DESIGN
├── 15-AI-System-Design/              # LLM Backends, Vector DBs, RAG (2026 Hot Topic)
│
└── 🏁 FINAL PREPARATION
    └── 16-Interview-Simulation/      # Mock Rubrics and Follow-up Questions
```

---

## 🚦 Getting Started

1.  **Read the [AUDIT-AND-TRANSFORMATION-PLAN.md](AUDIT-AND-TRANSFORMATION-PLAN.md)**: Understand what most candidates get wrong.
2.  **Follow the [ROADMAP-12-WEEKS.md](ROADMAP-12-WEEKS.md)**: A day-by-day plan for deep prep.
3.  **Self-Evaluate using [PRACTICE-CHECKLIST.md](PRACTICE-CHECKLIST.md)**: Know your level (L5 vs L6).
4.  **Keep the [QUICK-REFERENCE.md](QUICK-REFERENCE.md) open** during all mock interviews.

---

## 🕵️ The "Interviewer's Secret Scorecard"
*What we actually write down while you're talking:*

| **Signal** | **What we look for** |
| :-- | :-- |
| **Clarification** | Does the candidate ask about "Out of Scope" features? |
| **Calculations** | Can they estimate storage for 1B users in their head? |
| **Tradeoffs** | "I use X for Y, accepting Z as a penalty." (Standard of Excellence) |
| **Failures** | "What happens when the Redis node dies?" (Zero to Hero) |
| **Pacing** | Do they finish the high-level design in < 15 minutes? |

---

## ✨ 2026 Special Topic: AI Systems
Large Language Model (LLM) system design is now a **standard** question at MAANG (L5+).
*   **Unit 15** covers: Vector Databases, RAG (Retrieval Augmented Generation), and Prompt Caching strategies. **Do not skip this.**

---

## 🤝 Contributing
Found a gap? We're missing 6 case studies. PRs that follow the **New Template** are prioritized. See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 📚 Recommended Reading
1.  **Designing Data-Intensive Applications** (Kleppmann) - *The Bible*
2.  **System Design Interview (Volumes 1 & 2)** (Alex Xu) - *The Practical Guide*
3.  **Google SRE Book** - *For Staff+ Reliability Thinking*

---

**Built with ❤️ by Senior Staff Engineers who want you to succeed.**
