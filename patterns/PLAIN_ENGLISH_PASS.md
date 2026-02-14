# Pattern: Plain English Pass

## Problem

The PM Context Compression pattern compresses technical detail into 7-item summaries. This helps — but it compresses without translating. A summary line like "RNGProvider + RandomStream Protocols extracted to `rng_protocol.py`, 19 files updated, zero regressions" is shorter than the full debrief, but it still requires technical literacy to understand what happened and why it matters.

For non-technical operators coordinating agent fleets, this is a barrier. The operator needs to understand:
- What the team is building and why
- Whether the work matters for the product's goals
- What went wrong and what it means for the next cycle

None of these require understanding Protocols, type annotations, or pipeline positions. But the current debrief format only speaks in those terms.

## Solution

Add a **Plain English Pass** to every agent-to-human communication. Before the technical dump, the agent answers three questions in non-technical language:

### The Three Questions

**1. What problem did this solve?**
Describe the problem as a user or operator would experience it. No code references, no schema names, no rule IDs.

> "The game's dice roller was wired directly into every part of the combat engine. If we ever wanted to test combat with predictable dice (for debugging or replay), we'd have to rewrite every file that rolls dice."

**2. What does it actually do?**
Describe the mechanism in everyday terms. Imagine explaining it to someone who doesn't program.

> "We created a standard 'dice rolling interface' — a contract that says 'anything that can roll dice works here.' Then we updated every part of the combat engine to ask for 'something that rolls dice' instead of asking for the specific dice roller. Now you can swap in a test roller, a replay roller, or any future roller without touching the combat code."

**3. Why should anyone care?**
Describe the user-facing or project-level impact. What's different now? What's possible that wasn't before?

> "Testing and replaying combat is now possible without modifying the combat engine. Future features like deterministic replay and combat simulation can plug in without touching existing code."

### Word Budget

The Plain English Pass has a hard cap: **150 words total** across all three questions. This forces genuine compression — the agent can't write a technical explanation and call it "plain english." If you need more than 150 words, you're including jargon.

## The Four-Pass Debrief

With the Plain English Pass, the debrief becomes a four-pass structure:

```
Pass 0: Plain English      — 150 words, non-technical, operator reads this
Pass 1: Full Context Dump  — everything, agent-readable archive
Pass 2: PM Summary         — 7 items max, compressed for PM bandwidth
Pass 3: Retrospective      — what went well, what didn't, operational judgment
```

**Who reads what:**
- The **operator** reads Pass 0 and skims Pass 2 action items
- The **PM** reads Pass 2 and digs into Pass 1 when needed
- **Future agents** read Pass 1 for full context
- **Auditors** read Pass 1 and Pass 3 for integrity verification

## Implementation

### Debrief Template Addition

Add this section at the top of every debrief, before the technical content:

```markdown
## Plain English Summary

**What problem did this solve?**
[1-2 sentences. No jargon. Describe the problem as a user would experience it.]

**What does it actually do?**
[1-2 sentences. No jargon. Describe the mechanism in everyday language.]

**Why should anyone care?**
[1-2 sentences. Describe the impact — what's different now, what's possible that wasn't.]
```

### Quality Check

The Plain English Pass fails if:
- It contains class names, function names, or file paths
- It references rule IDs (RV-001, CT-003, BL-017)
- It uses programming terms (Protocol, dataclass, schema, refactor, type annotation)
- It exceeds 150 words
- A non-programmer couldn't understand it

### Worked Examples

**WO-RNG-PROTOCOL-001 (RNG Protocol Extraction):**

> **What problem did this solve?**
> The game's dice roller was hardwired into every combat calculation. Testing with predictable dice or replaying past combats required modifying the combat engine itself.
>
> **What does it actually do?**
> Created a standard "dice rolling contract" and updated every combat calculation to use the contract instead of the specific dice roller. Any dice roller that follows the contract now works everywhere.
>
> **Why should anyone care?**
> Deterministic testing and combat replay are now possible without touching the combat engine. Future dice-related features plug in cleanly.

(62 words)

**WO-ROADMAP-AUDIT (H1 Gap Analysis):**

> **What problem did this solve?**
> The project has a roadmap and a batch of work orders, but nobody had checked whether the work orders actually cover what the roadmap says should be built — or whether research findings imply work that nobody has planned yet.
>
> **What does it actually do?**
> Cross-referenced every planned work order against the roadmap and the research findings. Found 3 pieces of missing work, 2 items that should be promoted to higher priority, and 1 work order whose scope needs verification.
>
> **Why should anyone care?**
> Without this check, two quality rules would ship that can never actually detect errors (they check a field that's always empty), and a critical data pipeline would remain disconnected.

(117 words)

## When to Use

- Every debrief (mandatory)
- Every session memo that reaches the operator
- Handoff documents when the next session's operator (not just the next agent) needs to understand context
- Project status updates, roadmap summaries, and any artifact that crosses the technical/non-technical boundary

## When NOT to Use

- Agent-to-agent dispatches (builders don't need plain english — they need precise technical specs)
- Internal working notes
- Research memos (the PM translates these; the researcher writes for a technical audience)

## Anti-Patterns

- **Jargon in disguise.** "Extracted a protocol for dependency injection" is not plain english. "Created a standard contract so parts can be swapped" is.
- **Skipping it because it's "obvious."** Obvious to whom? The builder knows what they did. The operator doesn't. The 150-word pass takes 2 minutes to write and saves 20 minutes of operator confusion.
- **Writing the plain english pass last.** It should be written first — while the agent still has the high-level picture in mind. Writing it after the technical dump tends to produce a summary of the dump rather than a genuine translation.
- **Exceeding the word budget.** 150 words is a constraint, not a guideline. If you need more, you're including technical detail that belongs in Pass 1 or Pass 2.
