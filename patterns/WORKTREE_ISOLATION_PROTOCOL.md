# Pattern: Worktree Isolation Protocol

## Problem

When multiple builder agents work in parallel, they share the same working directory. Agents that are scoped to non-overlapping files can still collide at the filesystem level: one agent's partial writes, an uncommitted index state, or a stash can corrupt another agent's checkout. Parallel work is not truly parallel if agents share a filesystem root.

The subtler failure mode: an agent reads a file that another in-flight agent has partially modified. The read succeeds. The data is silently wrong. No error fires.

**The failure pattern:**
- Two builders are dispatched simultaneously
- Agent A modifies files in `aidm/resolvers/`; Agent B modifies files in `aidm/schemas/`
- These are correctly non-overlapping scopes on paper
- Agent A does a mid-task `git stash` to test something; Agent B's `git status` now shows stash changes
- Agent B's debrief includes files it never touched
- The PM has to untangle whose change is whose

**Root cause:** File ownership in the work order only prevents *logical* collisions. It does not prevent *filesystem* collisions. Two processes sharing a `.git` directory are not isolated.

---

## Solution

### Rule

**Each active builder session gets its own git worktree. One worktree per active builder. No exceptions during parallel dispatch.**

A git worktree is a linked working directory that shares the object store with the main repository but has its own index, HEAD, and working files. Two worktrees can be on different branches simultaneously. Each agent works in its own worktree, commits to its own branch, and cannot touch another agent's files — at the filesystem level, not just at the work-order level.

### When Worktrees Are Required

Worktrees are required when any of the following apply:

- **Parallel builder dispatch** — two or more builder WOs are active simultaneously
- **Audit + build overlap** — an Anvil audit is in flight at the same time as a Chisel build
- **Multi-builder + shared utility** — any WO touches a shared utility file (schemas, base resolvers, central dispatcher) while another WO is also in flight
- **Long-running sessions** — any session expected to span multiple hours, during which other sessions may be dispatched

Worktrees are **not** required for:
- Sequential builds (one completes, next begins)
- PM-only sessions (no code changes)
- Researcher/auditor sessions that are read-only

### Naming Convention

Worktrees use a three-part name:

```
{seat}-{batch}-{wo}
```

Examples:
- `chisel-w-wo1` — Chisel builder, Batch W, WO1
- `chisel-w-wo2` — Chisel builder, Batch W, WO2
- `anvil-audit-003` — Anvil auditor, Audit WO 003

**Branch naming follows the same convention:**

```
build/{seat}-{batch}-{wo}
```

Examples:
- `build/chisel-w-wo1`
- `build/chisel-w-wo2`
- `build/anvil-audit-003`

This makes `git branch -a` readable at a glance during parallel dispatch.

### Worktree Setup (per dispatch)

The PM includes worktree instructions in the dispatch. The builder executes them before touching any project file:

```bash
# From the repository root
git worktree add ../{worktree-name} -b build/{seat}-{batch}-{wo}

# Confirm
git worktree list
```

Example for Chisel building Batch W WO1:

```bash
git worktree add ../dnd35-chisel-w-wo1 -b build/chisel-w-wo1
cd ../dnd35-chisel-w-wo1
```

The builder then works exclusively in the worktree directory. All reads, all writes, all commits happen in the worktree. The main working directory (`f:\DnD-3.5` or equivalent) is never touched during a parallel session.

---

## Dispatch Metadata

Every work order dispatched during parallel sessions MUST include a Worktree block:

```markdown
## Worktree Assignment

| Field | Value |
|-------|-------|
| Worktree Path | `../{worktree-name}` |
| Branch | `build/{seat}-{batch}-{wo}` |
| Owner Seat | {Chisel / Anvil} |
| Setup Command | `git worktree add ../{worktree-name} -b build/{seat}-{batch}-{wo}` |
| Teardown Command | See cleanup rule below |
```

If a WO does not include a Worktree Assignment block during parallel dispatch, the builder **halts and requests the metadata** before beginning. A missing worktree assignment is a dispatch defect.

---

## Implementation

### Assignment Protocol

1. PM assigns worktree name and branch name when writing the dispatch
2. Builder creates the worktree before reading any project files
3. Builder confirms `git worktree list` shows the new worktree before proceeding
4. All work happens inside the worktree directory
5. Builder commits inside the worktree (`git commit` from the worktree path)
6. Debrief includes the commit hash from the worktree branch

### Cleanup Rule

After PM accepts a WO debrief (ACCEPTED verdict):

1. **Merge the worktree branch into main** (or the project's integration branch)
2. **Remove the worktree:** `git worktree remove ../{worktree-name}`
3. **Delete the branch:** `git branch -d build/{seat}-{batch}-{wo}`
4. PM confirms worktree is removed before dispatching the next round to the same seat

If a WO is **rejected**, the worktree is kept until the fix is delivered. The builder continues to work in the same worktree for the fix WO.

If a WO is **abandoned** (superseded or cancelled), the worktree is removed without merging: `git worktree remove --force ../{worktree-name}`.

### Handoff Rule

Worktrees belong to a session, not to a seat. A new agent session starting work on an incomplete WO should:

1. Check `git worktree list` from the repository root
2. If the worktree still exists: `cd` into it and continue
3. If the worktree was removed before the WO completed: recreate it from the debrief's last commit hash using `git worktree add ../{name} {commit-hash}`

A worktree is **never** handed to a different active builder while another builder's WO is in flight. Worktree reuse is sequential, not parallel.

### Conflict Rule

> **No shared worktree across active builders.**

If two builders are in flight simultaneously, they must be in separate worktrees on separate branches. A worktree that has been handed off to a new session is fine. A worktree shared between two simultaneous sessions is not allowed.

If the PM accidentally dispatches two WOs to the same worktree simultaneously, both builders halt and notify the PM. The PM resolves the collision — typically by creating a second worktree for one of the WOs.

---

## Debrief Requirements

The builder debrief must include:

1. **Worktree path** — confirms the correct worktree was used
2. **Branch name** — confirms commits went to the correct branch
3. **`git worktree list` output** — snapshot showing active worktrees at debrief time
4. **Commit hash** — from the worktree branch (not from main)

Missing any of these from the debrief during a parallel session = debrief **INCOMPLETE**.

---

## Real Example

During Batch W parallel dispatch, three builders were simultaneously in flight on W-WO1, W-WO2, and W-WO3. Without worktree isolation, each agent's `git status` would show uncommitted changes from the other agents. The stash from W-WO2's mid-session experiment appeared in W-WO1's debrief as "files modified." The PM had to manually verify which changes belonged to which WO.

**Without worktree isolation:** Three agents' states bleed together. Debrief verification requires manually parsing `git diff` across the full repo to assign changes to WOs.

**With worktree isolation:** Each agent's `git status` and `git diff` shows only its own changes. The debrief commit hash is definitive. Merge is a clean fast-forward. The PM's verification load drops to reading the debrief, not reconstructing it.

---

## Anti-Patterns

- **Sending two builders to the same worktree.** This is file ownership documented in the WO but not enforced at the filesystem level. You're back to the original problem.
- **Creating the worktree after starting work.** The agent reads files in the main directory, then moves to a worktree. The reads were in main, the writes are in the worktree. The state is now inconsistent.
- **Forgetting to remove worktrees.** `git worktree list` grows without bound. When it hits 5+ entries you lose visibility into which sessions are actually in flight.
- **Naming worktrees generically** (e.g., `temp1`, `temp2`). Opaque names make `git worktree list` unreadable during parallel dispatch. The `seat-batch-wo` convention makes the list self-documenting.
- **Not including the worktree block in the dispatch.** The builder will work in the main directory because it was never told not to. The dispatch is the contract; if the worktree isn't in the contract, it won't be used.

---

## When to Use

- Any parallel dispatch (2+ active builder or auditor WOs)
- Any session where audit and build overlap
- Any session where a shared utility is being modified concurrently with feature work

## When NOT to Use

- Sequential builds where each WO fully completes before the next begins
- Single-agent sessions with no parallel work in flight
- PM-only or researcher-only sessions (read-only work needs no filesystem isolation)
