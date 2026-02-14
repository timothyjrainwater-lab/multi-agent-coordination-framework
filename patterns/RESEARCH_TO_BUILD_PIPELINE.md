# Pattern: Research-to-Build Pipeline

## Problem

A human operator has an insight: "We should add voice input to the system." This is valuable but not actionable. If you dispatch a builder WO that says "add voice input," you get scope bleed, rework, and wasted context windows — because the builder has to make dozens of design decisions that should have been resolved before they started coding.

The gap between "raw insight" and "executable work order" is larger than it appears. Insights contain ambiguity, unexplored alternatives, and implicit assumptions. Builders need binary specs — unambiguous instructions with all decisions pre-resolved. Something has to convert the first into the second.

## Solution

A staged pipeline that converts raw operator insight into builder-ready work orders through explicit research and normalization steps. Each stage has a defined input, output, and responsible role.

### The Pipeline

```
BURST (raw insight)
  → RESEARCH WO (scoped question)
    → FINDINGS MEMO (research output)
      → BRICK (normalized, decision-ready packet)
        → BUILDER WO (executable spec)
```

### Stage 1: Burst

**What it is:** A raw insight from the operator. Unstructured, potentially vague, possibly brilliant. Not yet actionable.

**Examples:**
- "Speech input should feel reliable, not experimental"
- "Players need to see the tactical grid before casting spells"
- "The workflow keeps stuttering at handoff points"

**Where it lives:** Intake queue (a parking lot file). Bursts are captured immediately so they aren't lost, but they don't enter the production pipeline until converted.

**Who owns it:** The operator generates it. The PM captures and parks it.

### Stage 2: Research WO

**What it is:** The PM converts the burst into a scoped research question and drafts a research WO for a researcher agent to execute.

**Key properties:**
- The research question is specific and bounded (not "figure out voice input" but "what is the cold start latency of the TTS engine and what are the options for reducing it?")
- The research WO references specific files, schemas, or systems to examine
- The expected output format is defined (findings memo with recommendations)

**Who owns it:** PM drafts, operator approves dispatch, researcher executes.

### Stage 3: Findings Memo

**What it is:** The researcher's output. Contains findings, analysis, alternatives considered, and prioritized recommendations.

**Key properties:**
- Written for the PM, not for builders
- May contain multiple approaches with tradeoffs
- May identify binary decisions the operator must resolve
- May recommend follow-up research

**Who owns it:** Researcher produces, PM consumes.

### Stage 4: Brick

**What it is:** The PM normalizes the research findings into a READY packet with four components:

1. **Target Lock:** One sentence describing the end state. ("Speech input becomes deterministic, confirm-gated, and measurable.")
2. **Binary Decisions:** Enumerated choices the operator must resolve before builder WOs can be drafted. Each decision has exactly two options with clearly stated tradeoffs.
3. **Contract Spec:** The technical specification derived from research, written as a reference document the builder WO will point to.
4. **Implementation Plan:** Ordered list of builder WOs with dependencies and parallel opportunities.

**Key properties:**
- A Brick is READY when all four components exist
- Binary decisions must be resolved by the operator before proceeding
- The implementation plan may contain 1 WO or 20 — the research determines the scope

**Who owns it:** PM produces, operator resolves binary decisions.

### Stage 5: Builder WO

**What it is:** A self-contained work order drafted from the Brick. The builder never sees the upstream research.

**Key properties:**
- References the contract spec (from the Brick), not the research memos
- All design decisions are pre-resolved — the builder executes, doesn't decide
- Follows the standard Dispatch Self-Containment pattern

**Who owns it:** PM drafts from Brick, operator approves dispatch, builder executes.

### Why Builders Never See Research

Research memos contain:
- Multiple approaches with tradeoffs (the builder shouldn't re-evaluate — the PM already chose)
- Rejected alternatives (reading about rejected approaches wastes builder context)
- Nuance and uncertainty (builders need certainty — "do X", not "X or Y depending on...")
- Technical exploration that doesn't map to implementation steps

The Brick is the firewall. The PM extracts what the builder needs and discards the rest. This isn't information hiding — it's context window optimization.

## WIP Limits

- **1-2 READY Bricks ahead of the builder queue.** Don't stockpile Bricks — they go stale as the codebase evolves.
- **Don't open new research until current Bricks are converted or deprioritized.** Research without downstream conversion is waste.
- **One burst at a time through the pipeline.** Parallel research is fine (different questions, different researchers). Parallel conversion of the same burst's findings is not (it creates conflicting Bricks).

## Implementation

### Intake Queue Format

```markdown
# Burst Intake Queue

## READY Bricks
### BURST-001: [Title] — READY BRICK
**Target Lock:** [one sentence]
**Binary Decisions:** [list, with resolution status]
**Contract Spec:** [link to spec document]
**Implementation Plan:** [list of planned WOs]

## Active Bursts
### BURST-002: [Title]
**Target Lock:** [one sentence]
**Status:** [NOT STARTED | RESEARCH IN PROGRESS | AWAITING NORMALIZATION]

## Completed Bursts
[Archived after all builder WOs ship]
```

### Pipeline Tracking

Each burst moves through visible stages:

```
BURST-001: "Voice reliability"
  Stage 1 (Burst):    CAPTURED     — 2026-02-10
  Stage 2 (Research):  COMPLETE     — 5 research WOs executed
  Stage 3 (Findings):  COMPLETE     — Playbook synthesized
  Stage 4 (Brick):     READY        — 5 binary decisions awaiting operator
  Stage 5 (Builder):   NOT STARTED  — 19 WOs planned, pending DC resolution
```

## When to Use

- Any feature or system change that involves design decisions (not just implementation)
- Any work that requires understanding tradeoffs before coding
- Any operator insight that spans multiple WOs
- When the operator says "we should..." or "what if we..." — that's a burst

## When NOT to Use

- Bug fixes with obvious solutions (skip straight to builder WO)
- Mechanical refactors where the approach is predetermined (e.g., "extract this class into a protocol")
- Single-file changes with no design ambiguity

## Real Example

**BURST-001: Voice-First Reliability Membrane**

The operator's insight: "Speech input should feel reliable, not experimental."

Pipeline execution:
1. **Burst captured** — parked in intake queue
2. **5 Research WOs drafted and executed:**
   - WO-VOICE-RESEARCH-01: Voice Control Plane Contract
   - WO-VOICE-RESEARCH-02: Failure Taxonomy & Unknown Policy
   - WO-VOICE-RESEARCH-03: Metrics & Telemetry Spec
   - WO-VOICE-RESEARCH-04: UX Turn-Taking & Confirmation
   - WO-VOICE-RESEARCH-05: Synthesis Playbook
3. **Findings normalized into Brick:**
   - Target Lock: "Speech input becomes deterministic, confirm-gated, and measurable"
   - 5 binary decisions identified for operator resolution
   - Contract spec: 547-line playbook covering control plane, failure policy, metrics, prosody
   - Implementation plan: 19 builder WOs across 5 tiers
4. **Binary decisions presented to operator** — awaiting resolution before builder WOs are drafted

Total research output: ~2,000 lines across 5 memos. Total builder input: self-contained WOs referencing a single contract spec. The builder never reads the 2,000 lines of research.

## Anti-Patterns

- **Skipping research and going straight to builder WOs.** Works for simple features. Produces rework for anything with design ambiguity. The research cost is paid either way — the question is whether you pay it in a focused research session or scattered across a builder session that keeps stopping to make design decisions.
- **Builders reading research memos.** Wastes context. The builder's job is to implement a spec, not to evaluate research. If the builder needs information from the research, it should be in the Brick or the WO.
- **PM stockpiling Bricks.** Bricks go stale as the codebase evolves. A Brick drafted from research done 2 weeks ago may reference code that's been refactored. Keep the pipeline short: research → Brick → build within the same project cycle.
- **Skipping binary decisions.** If the operator doesn't resolve the decisions, the PM guesses. PM guesses produce WOs that get reverted when the operator disagrees. The binary decision gate exists to prevent this.
