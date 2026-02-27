# Pattern: Parallel Implementation Parity

## Problem

A codebase accumulates **parallel code paths** — two or more functions that compute the same logical result independently. This is common in systems that grow incrementally: a "fast path" and a "full path," a simplified resolver and a complex one, a legacy handler and a new one.

When a work order targets one of these paths, it fixes that path. The other paths are not mentioned in the WO, so the builder doesn't know they exist. The result: the fix lands in one location but the same bug silently persists in the others.

**The root cause is that WO dispatches don't enumerate parallel paths.** Each WO is written to fix a specific known gap. It treats the target function as the only implementation. No one is responsible for asking "where else does this same logic exist?"

**The scale of the damage:** In one project, 21 independent modifier calculations drifted across two parallel code paths over 30+ work orders. Each individual WO was correct for its target path. No WO caused the drift. The drift accumulated silently because no dispatch ever listed both paths. The divergence was only discovered by a systematic audit 30+ work orders later — representing significant rework and regression risk.

## Solution

### Rule

**Every WO that modifies a function that computes a result must identify ALL parallel code paths that compute the same result before work begins.**

The builder verifies parity across all identified paths before filing the debrief. A debrief that doesn't address all parallel paths is incomplete.

### How to Find Parallel Paths

Before drafting a WO, ask:

1. Is there more than one function or code path that produces this output?
2. Are there "light" and "heavy" variants of the same operation? (e.g., single-item and batch processors, simplified and full-featured versions)
3. Are there legacy code paths that handle the same case?
4. Does the same calculation appear in both "read" and "write" paths?

Document all paths in the WO's **Parallel Implementation Paths** section. If you don't know all paths, write: *"Builder: identify parallel paths as first task before any code changes."*

### Dispatch Template Section

Add this section to every WO that modifies resolver/calculator functions:

```markdown
## Parallel Implementation Paths

| Path | File | Function | Status |
|------|------|----------|--------|
| Primary | [file.py] | [function_name()] | This WO targets |
| Secondary | [file.py] | [function_name()] | Builder must verify parity |
| Legacy | [file.py] | [function_name()] | Builder must verify parity |

**Builder instructions:** Verify that all parallel paths produce the same result for the same inputs after this WO lands. If they don't, the secondary paths must be updated in the same commit or the divergence must be logged as a finding.
```

### Parity Verification

The builder must confirm one of the following for each parallel path:

- **Same fix applied** — the same correction was applied to this path
- **Already correct** — this path was already correct; no change needed; verified by inspection
- **Delegates** — this path delegates to the primary path; no independent implementation
- **Scoped out** — this path is intentionally different (document why)

The debrief must include a parity table:

```markdown
## Parallel Path Parity

| Path | File:Line | Outcome | Notes |
|------|-----------|---------|-------|
| Primary | [file:line] | Fixed | This WO |
| Secondary | [file:line] | Already correct | Delegates to primary |
| Legacy | [file:line] | Fixed | Same change applied |
```

Missing parity table → debrief rejected.

## When the PM Doesn't Know the Parallel Paths

This is normal. The PM drafts WOs from findings and research, not from full codebase knowledge. When the parallel paths are unknown at dispatch time, the WO dispatch section reads:

> *"Builder: identify all parallel paths implementing [logic description] as your first task. Document them before writing any code. If parallel paths exist and are not addressed, log a finding."*

The builder has context the PM doesn't. Making parallel path identification the builder's first task puts the responsibility at the right point in the workflow — with the agent that can actually verify it.

## Implementation

### Checklist: Does This WO Need Parity Verification?

| If the WO... | Then parity verification is... |
|--------------|--------------------------------|
| Modifies a function that computes a score, value, or result | Required |
| Modifies a "fast path" that also has a "full path" | Required |
| Modifies a shared utility function | Required (check all callers) |
| Adds a new modifier, bonus, or penalty | Required (find all paths that apply modifiers) |
| Modifies a single-item processor with a batch equivalent | Required |
| Adds a new field to a data structure | Required (check all read sites) |
| Fixes a display bug | Usually not required |
| Updates documentation | Not required |

### Anti-pattern: The "I Only Fixed One" Debrief

A debrief that reads *"Fixed the calculation in [function A]. Tests pass."* without mentioning parallel paths is the anti-pattern. The builder may have legitimately checked and found no parallel paths — but the debrief must say so:

> *"Searched for parallel implementations of [logic]. Found none. Only [function A] implements this calculation."*

This one-sentence confirmation closes the audit gap without requiring parity work when there's nothing to verify.

## Why This Is Hard to Detect Without the Pattern

Unit tests don't catch parallel path drift. They test the paths they cover. If a test covers path A and another test covers path B, and both tests pass, the tests say nothing about whether A and B produce the same result. The divergence is invisible until a cross-path integration test, a systematic audit, or a production incident surfaces it.

Parallel path drift is a **consistency failure that hides behind correct per-path tests.** The only prevention is explicitly tracking which paths are parallel, which is a process intervention, not a testing intervention.

## When to Use

- Any project where the same logical operation has more than one implementation
- Any project where code has a "simple" and "complex" version of the same function
- Any project that has grown incrementally (every large project eventually has parallel paths)

## Real Example

A project's result calculation had a "single-item" resolver and a "batch" resolver. The single-item resolver was the original implementation. The batch resolver was added later to support a new feature — it reproduced the calculation logic independently rather than delegating.

Over 30+ work orders, every WO that improved the calculation targeted the single-item resolver. The batch resolver drifted. A systematic audit eventually found that 21 independent calculation components had diverged between the two paths. None of the individual WOs were wrong — no one knew to look at the batch path.

**Fix:** Added a "Parallel Implementation Paths" section to every dispatch template targeting calculation functions. The batch resolver was refactored to delegate to the single-item resolver, eliminating the divergence permanently.
