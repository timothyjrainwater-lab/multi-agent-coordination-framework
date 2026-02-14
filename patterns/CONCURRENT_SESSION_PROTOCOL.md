# Concurrent Session Protocol

## Problem

You have two agent sessions running in parallel. Both need to edit files. If they edit the same file, the second commit overwrites the first. If they edit different files that reference the same facts, you get cross-file inconsistency. If they both run tests, they may interfere with each other's test fixtures.

Multi-agent parallelism is powerful — the D&D 3.5e project dispatched 7 parallel agent groups to fix 30 bugs simultaneously. But parallelism without file ownership produces collisions that are worse than doing things sequentially.

## Solution

**Partition work by file ownership.** Before dispatching parallel sessions, identify which files each session will touch. If two sessions need the same file, they must run sequentially, not in parallel.

### File Ownership Analysis

Before parallel dispatch, build a file-to-session map:

```
Session A: conditions.py, attack_resolver.py
Session B: maneuver_resolver.py
Session C: terrain_resolver.py, mounted_combat.py
Session D: spell_resolver.py, play_loop.py
```

**Conflict check:** Do any files appear in more than one session? If yes, those sessions must be sequenced.

```
CONFLICT: attack_resolver.py appears in Session A and Session E
RESOLUTION: Session E runs AFTER Session A completes
```

### Dependency Declaration

Each work order dispatch must declare:
- **Files I will modify** (write set)
- **Files I will read but not modify** (read set)
- **Files that must be in a specific state before I start** (dependency set)

Write-write conflicts require sequencing. Read-write conflicts are acceptable if the reader goes first or the reader tolerates the write.

### Implementation

**1. Pre-dispatch conflict check**

The coordinator (human or PM agent) reviews all parallel dispatches and identifies file overlaps. This is a manual step — the cost of getting it wrong (merge conflicts, silent overwrites) is much higher than the cost of spending 2 minutes checking.

**2. Session scope declaration**

Each agent's dispatch includes a scope block:

```markdown
## Session Scope
**Write set:** aidm/core/maneuver_resolver.py, aidm/schemas/maneuvers.py
**Read set:** aidm/schemas/attack.py, aidm/core/attack_resolver.py
**Dependencies:** WO-FIX-03 must be committed before this session starts
**Do NOT touch:** Any file not listed above
```

**3. Post-session verification**

After all parallel sessions complete, run `git diff --stat` to verify only the declared files were modified. If an agent touched a file outside its write set, investigate before committing.

## When to Use

- Any time you dispatch 2+ agent sessions simultaneously
- When a work order has explicit dependencies on another WO's output
- When multiple WOs touch the same subsystem (even different files — they may share helper functions)

## When NOT to Use

- Sequential single-agent work (no conflict possible)
- Read-only tasks like verification or research (no writes, no conflicts)

## Real Example

The fix WO dispatch for the D&D 3.5e project identified these file overlaps:

| File | Used by WOs |
|------|------------|
| `attack_resolver.py` | WO-FIX-01, WO-FIX-03 |
| `full_attack_resolver.py` | WO-FIX-01, WO-FIX-02, WO-FIX-03 |
| `conditions.py` | WO-FIX-03, WO-FIX-04 |
| `maneuver_resolver.py` | WO-FIX-07, WO-FIX-08, WO-FIX-09 |
| `play_loop.py` | WO-FIX-06, WO-FIX-11 |

This analysis produced 7 parallel-safe groups instead of 13 independent dispatches. Groups with file overlap ran their WOs sequentially within the group. Groups without overlap ran in parallel.

**Result:** 7 parallel agents completed 12 WOs. No merge conflicts. But 3 agents silently failed to write changes — a different failure mode that the concurrent protocol doesn't address (see: Silent Completion failure pattern).

## Anti-Patterns

- **"Just let them both edit and merge later"** — LLM agents don't produce clean diffs. Merging two agents' independent edits to the same file is manual reconstruction, not automated merging.
- **Declaring file ownership but not enforcing it** — if the dispatch says "do NOT touch other files" but nothing checks, agents will touch other files. The post-session `git diff --stat` is the enforcement.
- **Assuming read-only operations are safe in parallel** — they usually are, but if a reader caches assumptions from a file that a parallel writer is changing, the reader's output may be based on a state that no longer exists by the time it's used.
