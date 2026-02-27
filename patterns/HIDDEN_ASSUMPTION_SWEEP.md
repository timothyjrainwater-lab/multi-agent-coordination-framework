# Pattern: Hidden Assumption Sweep

## Problem

Requirements documents and specifications are written for human practitioners who already carry undocumented layers of judgment, context, and common sense. When you build a system from those documents, those hidden layers do not transfer automatically — they surface later as unexpected gaps in behavior.

This creates a specific failure mode: a seemingly simple question reveals that a fundamental assumption about scope or behavior was wrong. Left unaddressed, these moments become silent production failures.

**The failure pattern:**
- Someone notices an edge case or asks an obvious question
- The question turns out not to be answerable from existing specs
- Rather than stopping, the team makes a locally-reasonable assumption and continues
- The assumption bakes into implementation
- Weeks later, the assumption is discovered to have been wrong
- Rework follows

**Root cause:** There is no protocol for catching these moments at the right time. The question felt small. Nobody classified it as architectural. It slipped through.

---

## Solution

### Rule

**When a question starts to feel larger than it should, run the Hidden Assumption Sweep before continuing.**

The sweep is a 10-question triage protocol. It takes 3-10 minutes. Its goal is not to solve the problem — it is to classify it correctly so it gets the right kind of follow-up.

**Classification rule:**

> If the answer changes what "done" means, crosses a system boundary, or reveals that the system currently fails open — file it as a strategy item, not a work order.

### The 10-Question Sweep

Answer each question YES / NO / UNKNOWN.

1. **Is this a missing feature or a missing assumption?** (Assumptions are more dangerous — they affect everything built on top of them)
2. **Does it change what "done" means for any existing WO or milestone?**
3. **Does it live at a system boundary?** (Between two subsystems, between system and users, between two resolvers that hand off to each other)
4. **Does the system currently fail closed (silent, safe) or fail open (inventing answers) at this point?**
5. **Does it require a human judgment layer that the source documentation never explicitly defines?**
6. **Does it affect multiple subsystems or just one?**
7. **Can a user or adversary expose this gap through normal, reasonable behavior?**
8. **Is there an observable failure mode, or is this a silent failure?**
9. **Who is authorized to make the ruling if the system cannot?**
10. **What must be true about system state for any ruling here to be meaningful?** (If the required state is not tracked, flag as a world-model gap — not just a logic gap)

### Escalation Rule

**Escalate to STRATEGY (not WO) if:**
- 3 or more answers are UNKNOWN, **or**
- the issue changes the definition of done for anything in progress, **or**
- the issue crosses a system boundary where current behavior is not fail-closed

When in doubt, escalate. The cost of filing a strategy item that turns out to be minor is low. The cost of treating an architectural assumption as a routine work order is high.

### Grenade Categories (Quick Reference)

| Category | Description |
|----------|-------------|
| **Hidden Human Layer** | Judgment, pacing, or common sense that practitioners know implicitly but the spec never states |
| **Boundary Protocol Gap** | The handoff between two subsystems is undefined or ambiguous |
| **Fail-Open Gap** | Uncertainty causes the system to invent answers rather than surface the uncertainty |
| **Observability Gap** | No way to know what the system decided or why |
| **Definition-of-Done Drift** | Completion criteria have changed but existing WOs have not been updated |
| **Authority Gap** | Who makes the ruling when the system cannot? Nobody knows. |
| **Promotion Path Gap** | One-off rulings made ad hoc never become documented, repeatable capability |

---

## Implementation

### When to Run the Sweep

Run immediately when any of the following happens:

- A small idea suddenly reframes architecture or scope
- A fix starts to feel too large for a normal work order
- Something feels dangerous but you cannot yet name why
- A discussion shifts from "how to implement this" to "what does done actually mean"
- A test case reveals behavior that is technically correct but clearly wrong in context

**Target runtime:** 3-10 minutes for first pass. You are classifying, not solving.

### Output Format

Every sweep produces a short artifact note:

```markdown
**Working name:** [what this issue is, in plain language]
**Category:** [from the grenade categories above, or describe]
**Type:** assumption gap | boundary gap | DoD correction | missing protocol
**Immediate risk:** [what can go wrong if this is ignored]
**Current behavior:** known | unknown | fail-open | fail-closed
**Proposed artifact type:** strategy | probe | WO | gate | spec | design note
**Blocking status:** blocking | non-blocking | shadow-track
```

This artifact is filed immediately — not noted for later, not mentioned in a message. Filed.

### Routing After Classification

| Classification | Action |
|----------------|--------|
| Feature-local issue | Draft a WO, scope contained, gate as usual |
| Missing assumption or boundary issue | File STRATEGY first, then derive supporting WOs |
| Unknown risk or meaningful concern | File a PROBE before implementation direction |
| Vague or unverifiable criteria | File a GATE or acceptance rubric first |
| Reframes architecture or doctrine | File a DESIGN NOTE or THESIS anchor |

Strategy items are not resolved by the PM alone. The operator reviews and makes explicit decisions. Strategy items resolved in conversation without an artifact violate the protocol.

### The Fail-Safe

If you cannot classify the issue confidently in one pass:
- Mark it UNKNOWN / STRATEGY CANDIDATE
- File a short note
- Hand to PM for packetization
- Do not carry the full problem in working memory

The point of the protocol is to prevent operator overload from becoming architecture drift.

---

## When to Use

- Any project where requirements come from an external specification written for human practitioners
- Any project where "reasonable interpretation" has historically meant different things to different agents
- Any time a question arises that starts with "obviously the system would" — test that assumption
- Before any cross-cutting change that touches a shared utility or central coordinator

## When NOT to Use

- For clearly scoped, local feature requests with no cross-system implications
- When the answer is unambiguously in the specification and no judgment is required
- As a substitute for a proper design review on a major feature (the sweep classifies, it does not replace design)

---

## Real Example

A team was building an engine that resolved named game mechanics from a rulebook. During a build session, someone asked: what happens when a player tries to do something the rulebook does not have a rule for?

The question felt small. The immediate instinct was to add a fallback handler work order.

Running the sweep revealed:
- Q5 (human judgment layer): YES — the rulebook assumes a human referee who can reason by analogy
- Q4 (fail-open): YES — the current system would invent a result rather than surface the uncertainty
- Q10 (required state): UNKNOWN — the system does not track enough scene context to reason about improvised actions
- Classification: 4 UNKNOWNs — escalate to STRATEGY

The STRATEGY item led to a design document identifying three architectural options. The builder WO that was almost dispatched would have solved the symptom while missing the architecture question entirely.

**Without the sweep:** A patch would have shipped. The underlying gap would have accumulated silently.
**With the sweep:** The question got the right level of attention. The architecture decision was explicit. Subsequent WOs were written against a clear spec.

---

## Anti-Patterns

- **Filing a WO for something that returned UNKNOWN on 3+ sweep questions.** That is a strategy item wearing a WO's clothes.
- **Running the sweep without filing the output.** The sweep produces an artifact, not a conversation.
- **Using the sweep to defer indefinitely.** Classify, route, act. The sweep is a triage tool, not a parking lot.
- **Treating "nobody else noticed" as evidence it is not a real gap.** Hidden assumptions are hidden precisely because they feel obvious until they are not.
