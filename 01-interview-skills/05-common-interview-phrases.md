# Common Interview Phrases

## 1. Problem Statement

What phrases should you use in system design interviews to demonstrate senior engineering thinking?

---

## 2. Opening Phrases

### Restating the Problem
* "Let me make sure I understand correctly..."
* "So we're designing [system] that needs to [goal]?"
* "Just to clarify, the main requirement is..."

### Setting Expectations
* "I'll start with a high-level design, then we can dive deeper"
* "Let me think through this step by step"
* "I'll make some assumptions, and we can adjust as needed"

---

## 3. Clarifying Questions

### Scale Questions
* "What's the expected scale in terms of users and QPS?"
* "Are we talking about millions or billions of users?"
* "What's the peak vs average traffic?"

### Feature Questions
* "What features are must-haves vs nice-to-haves?"
* "Is [feature] required, or can we start without it?"
* "What's the priority order of features?"

### Constraint Questions
* "What are the latency and availability requirements?"
* "Are there any specific constraints I should be aware of?"
* "What's the acceptable trade-off between X and Y?"

---

## 4. Architecture Explanation

### High-Level
* "At a high level, I see [X] main components..."
* "The architecture consists of..."
* "Let me break this down into [layers/components]..."

### Component Details
* "For [component], I'll use [technology] because..."
* "This component handles..."
* "The data flows from [A] to [B] to [C]..."

### Scaling
* "To scale this, I would..."
* "The bottleneck will likely be..."
* "If we need to handle 10x traffic, we could..."

---

## 5. Trade-off Phrases

### Stating Decisions
* "I'm choosing [X] because [requirement]..."
* "For this use case, [X] is more important than [Y]..."
* "I'm making a trade-off here: [X] for [Y]..."

### Acknowledging Alternatives
* "The alternative would be [Y], but..."
* "I considered [X], but chose [Y] because..."
* "We could use [X] or [Y], but [Y] fits better because..."

### Explaining Trade-offs
* "The trade-off is [what you're giving up], but [why it's worth it]"
* "We're optimizing for [X] at the expense of [Y]..."
* "We accept [trade-off] because [justification]..."

---

## 6. Problem-Solving Phrases

### Identifying Issues
* "The bottleneck here will be..."
* "The challenge with this approach is..."
* "One thing to watch out for is..."

### Proposing Solutions
* "To handle this, I would..."
* "One way to address this is..."
* "We could solve this by..."

### Considering Edge Cases
* "What if [edge case] happens?"
* "We should also consider..."
* "Another scenario to handle is..."

---

## 7. Communication Phrases

### Checking Understanding
* "Does this make sense?"
* "Am I on the right track?"
* "Should I continue or dive deeper here?"

### Asking for Input
* "What do you think about this approach?"
* "Would you like me to explore [alternative]?"
* "Is there a specific aspect you'd like me to focus on?"

### Handling Uncertainty
* "I'm not 100% sure, but my thinking is..."
* "I'd need to verify this, but I believe..."
* "Let me think about that for a moment..."

---

## 8. Deep-Dive Phrases

### Going Deeper
* "Let me dive deeper into [component]..."
* "For [component], here's how it works..."
* "The implementation details are..."

### Simplifying
* "Let me simplify this..."
* "At its core, this is about..."
* "The key idea here is..."

---

## 9. Wrap-up Phrases

### Summarizing
* "To summarize, the key decisions are..."
* "The main trade-offs we made are..."
* "In essence, we're [doing X] to achieve [Y]..."

### Next Steps
* "If we had more time, I'd explore..."
* "The next steps would be to..."
* "To productionize this, we'd need to..."

---

## 10. Phrases to Avoid

### Weak Language
* ❌ "I guess we could..."
* ❌ "Maybe we should..."
* ❌ "I think this might work..."

### Better Alternatives
* ✅ "I would choose..."
* ✅ "We should use..."
* ✅ "This approach will work because..."

### Overconfidence
* ❌ "This is definitely the best way..."
* ❌ "There's no other option..."
* ❌ "This will definitely work..."

### Better Alternatives
* ✅ "I believe this is the right approach because..."
* ✅ "This seems like the best fit given the constraints..."
* ✅ "This should work, but we'd need to test..."

---

## 11. Example Full Explanation

> "Let me make sure I understand: we're designing a URL shortener that needs to handle 100M URLs per day with a 10:1 read/write ratio. The main requirements are low latency redirects and high availability.
>
> At a high level, I see three main components: an API service for creating and redirecting, a cache layer for hot URLs, and a database for persistence.
>
> I'm choosing NoSQL for the database because we need horizontal scalability for 15K writes per second. The trade-off is we lose ACID transactions, but for this use case, scalability is more important.
>
> For caching, I'll use Redis with a cache-aside pattern. This will give us sub-10ms redirects for cached URLs, which should cover 80% of traffic. The trade-off is we may serve slightly stale data, but URL updates are rare.
>
> Does this architecture make sense? Should I dive deeper into any component?"

---

## 12. Practice Tips

### Memorize Key Phrases
* Practice saying trade-off phrases
* Practice clarifying question formats
* Practice architecture explanation structure

### Use in Practice
* Record yourself using these phrases
* Get feedback on clarity
* Refine based on what works

### Adapt to Style
* Some interviewers prefer more detail
* Some prefer high-level
* Adjust based on interviewer's style
