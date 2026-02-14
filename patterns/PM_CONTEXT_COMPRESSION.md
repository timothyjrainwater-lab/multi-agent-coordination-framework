# PM Context Compression

## Problem

The human coordinator (PM) has the smallest context window in the system. An agent can read 200K tokens per session. A human scanning files between meetings might read 2K-5K tokens before context fatigue sets in. Every agent produces output. The PM must consume enough output to make good decisions without drowning in detail.

If agents produce verbose reports, the PM skips them. If agents produce terse summaries, the PM misses critical details. Both failure modes lead to bad decisions — either from information overload or information starvation.

## Solution

**Two-pass writing.** Every agent-to-PM communication follows the same structure: full dump first (for the agent's own completeness), then compressed summary (for the PM's bandwidth).

### The Two-Pass Protocol

**Pass 1: Full Debrief (agent reads, PM doesn't)**

The agent writes everything it knows — complete findings, full reasoning, all edge cases, raw data. This document is the artifact of record. If a future agent needs the details, they're here.

**Pass 2: PM Summary (PM reads)**

The agent re-reads its own debrief and compresses it to the minimum the PM needs to make decisions. Rules:

- **7 items maximum.** If you have more than 7 findings, prioritize.
- **One line per item.** Description + action needed. No background, no reasoning.
- **Action items first.** Things the PM must decide or approve go at the top.
- **Status items second.** Things the PM should know but doesn't need to act on.
- **Deferred items last.** Things that can wait.

### PM Briefing Structure

```markdown
## Requires PM Action
1. [Item] — [what PM must decide]
2. [Item] — [what PM must approve]

## Status (Informational)
3. [Item] — [what happened]
4. [Item] — [what changed]

## Deferred
5. [Item] — [can wait until X]
```

### Implementation

**Rolling briefing file.** Instead of the PM reading 23 individual files, maintain a single `PM_BRIEFING_CURRENT.md` that every agent updates when they add files. The PM reads one file, gets the full picture, and digs into specific documents only when needed.

**Session memo template.** Standardize the PM-facing output so every session produces the same structure. The PM learns the format once and can scan it in seconds.

## When to Use

- Every session that produces findings, status changes, or action items for the PM
- When an agent discovers something the PM needs to know about
- At the end of every work session (debrief protocol)

## When NOT to Use

- Agent-to-agent communication (use full-detail dispatches — agents have the context budget)
- Internal working notes (keep in session, don't surface to PM unless relevant)

## Real Example

A builder agent completed 12 fix work orders in a session. The full debrief covered: which agents failed silently (3/7), which WOs needed reclassification, schema additions to ActiveSpellEffect, gold master regeneration, test assertion updates, and a cascade analysis of WO-FIX-03.

The PM summary was 7 lines:

```
1. WO-FIX-11: RETIRE. Action cost table doesn't exist as described.
2. BUG-F2/F3: UNVERIFIED. Desk check leveling.py:291-308.
3. Agent failure rate: 3/7. Gate needed: git diff before accepting.
4. WO-FIX-03 cascade: WO missed conditions.py aggregation file.
5. ActiveSpellEffect gained spell_level field.
6. Gold masters regenerated (4 files) — expected.
7. NEW PROCESS: Builder writes full debrief, then compresses for PM.
```

The PM read 7 lines. The full debrief exists if anyone needs the details later.

## Anti-Patterns

- **Sending the full debrief to the PM** — they won't read it. They'll skim, miss action items, and make decisions on incomplete information.
- **Writing only the summary** — the details are lost. A future agent can't reconstruct what happened from 7 lines.
- **Mixing action items with status updates** — the PM needs to know "what do I need to do?" separately from "what happened?" If they're interleaved, action items get missed.
- **Assuming the PM will read the files you reference** — they might not. The summary must be self-contained enough to act on without reading anything else.
