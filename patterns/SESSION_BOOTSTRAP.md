# Session Bootstrap

## Problem

An agent starts a new session and reads the project's documentation to orient itself. The documentation says there are 5,100 tests. The agent proceeds on that assumption. But 3 tests were broken by last session's changes, and nobody updated the docs. The agent now has a false foundation — and every decision it makes from here may be wrong.

Prose documents are always potentially stale. The gap between "what the docs say" and "what the code actually does" widens with every session that touches code but doesn't update docs. Agents can't tell the difference between current truth and stale truth when both are written in the same authoritative tone.

## Solution

**Start every session with machine-verified truth.** Before reading any prose document, run commands that produce canonical facts about the project's current state.

### Bootstrap Sequence

```bash
# 1. What branch, what's changed, what's committed?
git status
git log --oneline -5

# 2. Do the tests pass? How many?
python -m pytest tests/ -q --tb=no 2>&1 | tail -3

# 3. What files were touched recently?
git diff --stat HEAD~5
```

This takes 30 seconds and produces ground truth that no prose document can override.

### Implementation

**Option A: Manual bootstrap (Tier 3)**

Add to the onboarding checklist: "Before reading any docs, run these 3 commands and report the output."

**Option B: Bootstrap script (Tier 1.5)**

```bash
#!/bin/bash
# scripts/session_bootstrap.sh
echo "=== SESSION BOOTSTRAP ==="
echo "--- Branch & Status ---"
git status --short
echo "--- Recent Commits ---"
git log --oneline -5
echo "--- Test Health ---"
python -m pytest tests/ -q --tb=no 2>&1 | tail -3
echo "--- Uncommitted Changes ---"
git diff --stat
echo "=== END BOOTSTRAP ==="
```

The agent runs this first, then reads prose documents with the machine truth as a reference frame.

**Option C: Auto-generated state file (Tier 1)**

A CI step or pre-session hook generates `PROJECT_STATE_MACHINE.md` from actual project state. Agents read this file instead of manually-maintained state documents. The file is always regenerated, never edited by hand.

## When to Use

- Every new agent session, without exception
- After pulling changes from a remote
- After another agent's session ends and before the next begins
- When an agent reports something that contradicts what you expect (re-bootstrap to check)

## When NOT to Use

- Mid-session for routine work (the bootstrap established truth at session start; re-running mid-session wastes context)
- As a substitute for reading project documentation (bootstrap establishes facts; docs explain architecture and intent)

## Real Example

The D&D 3.5e project's `PROJECT_STATE_DIGEST.md` claimed 303 formulas verified. After Domain A re-verification and Domain I expansion, the actual count was 338. An agent reading the PSD would operate on stale numbers. A session bootstrap running `grep -c "FORMULA:" docs/verification/DOMAIN_*` would produce the correct count immediately.

## Anti-Patterns

- **Trusting test count from a document** — run `pytest` and count the output. Documents lie by omission (they don't update themselves).
- **Skipping bootstrap because "I just read the docs"** — the docs were written by a previous agent that may have had stale information itself. The staleness compounds.
- **Running bootstrap but not reporting it** — the bootstrap output should be the first thing in the agent's response, so the human coordinator can verify it matches expectations.
