# How to Use This Playbook

> **Target**: L4-L7 candidates | **Time to Complete**: 2 hours reading + 12 weeks practice

---

## 1. The Philosophy

This playbook is built on a single insight: **MAANG system design interviews test decision-making, not memorization**.

The difference between a Hire and a Strong Hire:
- **Hire**: Knows technologies and patterns
- **Strong Hire**: Can justify WHY they chose one technology over another with quantified tradeoffs

---

## 2. Structure Overview

```
├── 00-Orientation/          # Start here: understand your level
├── 01-Interview-Skills/      # Communication, pacing, recovery
├── 02-Deep-Fundamentals/     # The "why": consensus, storage, clocks
├── 03-Building-Blocks/      # Components: LB, DB, Cache, Queue
├── 04-Patterns-and-Paradigms/ # Patterns: circuit breaker, sagas
├── 05-Decision-Tradeoffs/   # Tradeoff reasoning framework
├── 06-API-and-Data-Modeling/ # Schema design masterclass
├── 07-Case-Studies/          # 16 systems with full depth
├── 08-Diagrams/              # Mermaid + ASCII templates
├── 09-Advanced-Topics/       # Multi-region, chaos, DR
├── 10-Real-World-Failures/   # Postmortem analysis
├── 11-Scaling-Playbooks/     # Read-heavy, write-heavy, etc.
├── 12-Observability-and-SRE/ # Monitoring, SLOs, alerting
├── 13-Security-and-Compliance/ # Auth, encryption, GDPR
├── 14-Company-Specific/      # How FAANG actually builds
├── 15-AI-System-Design/      # LLM, vector DB, RAG
└── 16-Interview-Simulation/  # Mock interviews, rubrics
```

---

## 3. Level-Specific Path

### L4 (SDE II) - Junior
- Start: 00 → 01 → 02 → 03 → 07 (first 8 case studies)
- Goal: Complete any standard case study in 40 minutes

### L5 (Senior SDE)
- Start: Skip to 04 → 05 → 06 → 07 (all 16 case studies)
- Goal: Quantify tradeoffs, handle follow-ups

### L6 (Staff Engineer)
- Start: 09 → 10 → 12 → 13 → 14 → 16
- Goal: Multi-region design, cost modeling, organizational thinking

---

## 4. Daily Practice Routine

| Week | Daily Time | Activity |
|------|------------|----------|
| 1-3 | 1.5 hr | Read fundamentals, speak aloud |
| 4-6 | 1.5 hr | Build case studies, draw diagrams |
| 7-9 | 2 hr | Practice full 45-min sessions |
| 10-12 | 2 hr | Mock interviews, feedback review |

---

## 5. The 11-Step Framework

Every case study follows this structure:

1. **Clarify Requirements** - Don't assume, ask questions
2. **Scope the Problem** - Define what's IN and OUT
3. **Capacity Estimation** - QPS, storage, bandwidth math
4. **High-Level Design** - Draw the boxes
5. **API Design** - Define endpoints and contracts
6. **Data Model** - Schema with partition keys
7. **Deep Dive** - 2-3 components in detail
8. **Bottleneck Analysis** - What breaks at scale
9. **Failure Modes** - What happens when X fails
10. **Tradeoffs** - Explicit what's given up
11. **Monitoring** - Metrics, alerts, dashboards

---

## 6. How to Practice

### Step 1: Read (Weeks 1-6)
- Read each topic once, focus on understanding the "why"
- Take notes on tradeoffs (this is what interviewers test)

### Step 2: Speak Aloud (Weeks 7-9)
- Practice each case study speaking aloud for 45 minutes
- Record yourself, identify filler words ("um", "uh")
- Time your 30-second, 2-minute, 5-minute narratives

### Step 3: Mock Interviews (Weeks 10-12)
- Do 5+ full mock interviews
- Use the rubric in `16-Interview-Simulation/06-self-evaluation-rubric.md`
- Focus on recovery: "I don't know, but here's how I'd reason..."

---

## 7. The Staff Scorecard

Interviewers secretly evaluate:
1. **Scoping** - Do you define what's out of scope?
2. **Quantification** - Can you do back-of-envelope math?
3. **Failure Thinking** - Do you discuss failure modes proactively?
4. **Communication** - Do you check in ("Does this direction work?")?
5. **Depth on Demand** - Can you go 3 levels deeper on any topic?
6. **Intellectual Honesty** - Can you say "I don't know" gracefully?

---

## 8. Quick Reference Card

Keep `QUICK-REFERENCE.md` open during all practice:
- CAP theorem variants
- Common QPS calculations
- Database selection matrix
- Cache invalidation strategies
- Retry/backoff formulas

---

## 9. Contributing

Found a gap? PRs welcome. See `CONTRIBUTING.md` for:
- Case study template
- Diagram standards
- Level-calibration guidelines

---

## 10. The Final Test

Can you design this in 45 minutes with full confidence?

> **Design a system that ranks 1 billion ads per second across 3 continents with <50ms P99 latency**

If yes, you're ready for L6.
