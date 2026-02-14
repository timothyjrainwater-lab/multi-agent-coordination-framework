# Cross-File Consistency Gate

## Problem

When a fact changes — a verdict reclassification, a count update, a status change — it often lives in multiple files. An agent updates one file and moves on, leaving the others stale. The project now contains contradictory information, and the next agent to read both files has no way to know which is correct.

This is the most common coordination failure in multi-agent projects. It's silent (no test fails), cumulative (each partial update makes the state harder to reconstruct), and contagious (agents reading stale files propagate the error into new work).

## Solution

**All-or-nothing updates.** When a fact changes, the agent must update every file that references it in the same commit. If the agent can't identify all affected files, it must flag the update as partial and document what's missing.

### Implementation

**1. Cross-Reference Map**

Maintain a map of which facts live in which files:

```
FACT: Bug count per domain
FILES: BONE_LAYER_CHECKLIST.md (summary row)
       WRONG_VERDICTS_MASTER.md (row count)
       DOMAIN_X_VERIFICATION.md (section summary)
       FIX_WO_DISPATCH_PACKET.md (WO bug count)
```

When an agent changes a bug count, it must touch all four files.

**2. Consistency Gate in Commit Protocol**

Before committing a fact-change, the agent runs a mental (or scripted) check:

```
For each fact I changed:
  - Which other files reference this fact?
  - Did I update all of them?
  - If not, what's missing and why?
```

If any file is missed, add it to the same commit or document the gap explicitly.

**3. Tier 1 Enforcement (Optional)**

For critical facts (counts, statuses, version numbers), write a test that cross-checks files:

```python
def test_bug_count_consistency():
    """Checklist bug count must match WRONG_VERDICTS_MASTER row count."""
    checklist_count = parse_checklist_totals()
    master_count = count_master_rows()
    assert checklist_count == master_count
```

This catches drift automatically. The test name tells the fixing agent exactly what to reconcile.

## When to Use

- Any verdict, count, or status that appears in more than one file
- Schema changes that affect consumers (e.g., adding a field to a dataclass requires updating serialization, aggregation, and tests)
- Reclassifications (WRONG → AMBIGUOUS) that propagate across verification files, master lists, and dispatch packets

## When NOT to Use

- Single-file facts (a function's docstring describing its own behavior)
- Ephemeral state (a session memo's "current task" — it's only read once)

## Real Example

**Domain A re-verification** found 4 bug reclassifications. The agent updated the checklist and WRONG_VERDICTS_MASTER but missed DOMAIN_C_VERIFICATION.md. Result: the checklist said "1 WRONG / 3 AMBIGUOUS" for Domain C, but the verification file still said "3 WRONG / 1 AMBIGUOUS." A later agent reading the verification file would dispatch fix WOs for bugs that had already been reclassified as design decisions.

The fix: a second commit (`6982005`) caught the gap and brought the verification file into consistency. The lesson: the original commit should have touched all three files.

## Anti-Patterns

- **"I'll fix the other files later"** — you won't. The context rotates, the gap becomes invisible, and a future agent reads the stale file.
- **Updating only the "primary" file** — there is no primary file. All files that contain a fact are equally authoritative to the next agent that reads them.
- **Relying on agents to notice inconsistencies** — agents trust files. If a file says 3 WRONG, the agent believes 3 WRONG. The inconsistency is invisible unless both files are in the same context window.
