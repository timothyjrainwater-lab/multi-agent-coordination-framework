# Pattern: Debrief Integrity Boundary

## Problem

The debrief format trusts agents to accurately self-report what happened during a session — including mistakes, dead ends, and mid-session course corrections. This trust is mostly justified (agents don't have incentives to lie), but it creates a blind spot: the PM and operator cannot distinguish between "no concerns reported" meaning "the agent thought carefully and found nothing" versus "the agent didn't think about it."

The test suite provides partial verification — an agent can't fabricate test counts or claim files were modified when `git diff` shows otherwise. But the prose sections of a debrief (retrospective, methodology notes, concerns) are unverifiable by any automated means. The framework relies on agent honesty for these sections, and that reliance should be named explicitly rather than assumed implicitly.

## The Verification Spectrum

Not all debrief content has the same integrity level. Debrief fields fall on a spectrum from machine-verifiable to prose-only:

### Tier 1: Machine-Verifiable (Highest Integrity)

These fields can be independently confirmed by running a command. An agent cannot misrepresent them without the discrepancy being detectable.

| Field | Verification Method |
|-------|-------------------|
| Test pass/fail counts | `pytest` output |
| Files modified | `git diff --stat` |
| Commit hashes | `git log` |
| File existence | `ls` / `stat` |
| Import graph changes | `grep` for import statements |
| Boundary law compliance | Boundary law test suite |

**Integrity guarantee:** ~100%. If the agent claims 5,804 tests pass and `pytest` shows 5,800, the discrepancy is immediately visible.

### Tier 2: Process-Verifiable (Medium Integrity)

These fields can be cross-checked against other artifacts but require human judgment to interpret.

| Field | Verification Method |
|-------|-------------------|
| Files listed as modified | Cross-check against `git diff` |
| Scope compliance | Compare diff against WO scope boundaries |
| "What was done" summary | Compare against commit messages and diff |
| Mid-session corrections | Partial — visible in commit history if multiple commits |

**Integrity guarantee:** ~70-80%. The agent's description of what they did can be compared against what the diff shows, but the diff doesn't capture intent, reasoning, or rejected alternatives.

### Tier 3: Prose-Only (Lowest Integrity)

These fields exist only in the agent's self-report. No external artifact confirms or contradicts them.

| Field | What's Unverifiable |
|-------|-------------------|
| Retrospective | Whether the agent actually considered alternatives |
| Concerns section | Whether "no concerns" reflects analysis or omission |
| Methodology notes | Whether the described approach was actually followed |
| Fragility assessment | Whether the agent identified all fragile points |
| Process feedback | Whether the feedback reflects genuine evaluation |

**Integrity guarantee:** ~40-60%. The agent has no incentive to lie, but also no structural pressure to be thorough. An agent that runs out of context window will produce a thin retrospective, not because it's dishonest but because it's depleted.

## Mitigation Options

### Option 1: Cross-Agent Audit (Expensive)

A second agent reads the first agent's debrief alongside the actual diff and flags discrepancies. This catches cases where the debrief describes work that doesn't match the artifacts.

**Cost:** One additional agent session per debrief. Doubles the overhead.
**When to use:** High-stakes WOs where incorrect self-reporting could cascade (e.g., schema changes, security-sensitive code).

### Option 2: Spot-Check Protocol (Moderate)

The PM randomly selects one debrief per cycle and deep-dives: reads the diff, runs the tests, and compares against the debrief's claims. This provides statistical deterrence without auditing every session.

**Cost:** ~30 minutes of PM time per cycle.
**When to use:** Standard operating procedure. Catches systemic reporting gaps without auditing every session.

### Option 3: Expand the Machine-Verified Surface (Structural)

Add structured fields to the debrief that can be cross-checked:
- **Require `git diff --stat` output** in the debrief (already recommended in F-003 fix)
- **Require test output** (pass/fail/skip counts) as a structured field, not prose
- **Require commit hash list** so each claim of "I committed X" maps to a verifiable hash
- **Add a "files read" list** that can be cross-checked against the agent's actual tool usage (where platform supports it)

**Cost:** Template changes. Minimal per-session overhead.
**When to use:** Always. This is the lowest-cost, highest-impact mitigation.

### Option 4: Debrief-by-Next-Agent (Structural)

Instead of the builder writing their own debrief, the next agent writes the debrief by reading the diff and commit history. This provides natural cross-agent verification — a different agent interpreting what was done, rather than the builder self-reporting.

**Cost:** Small overhead at the start of the next session.
**When to use:** When builder context windows are depleted (debrief quality degrades at window end). Also addresses the structural problem of mandatory output (commit + debrief) coming at the worst possible time — when the window is most exhausted.

## Implementation

### Minimum Viable Integrity Gate

Add these structured fields to the debrief template (non-prose, machine-checkable):

```markdown
## Machine-Verified Output
- **Test results:** [paste pytest summary line]
- **Files modified:** [paste git diff --stat output]
- **Commits:** [list commit hashes with one-line messages]
- **Pre-existing failures:** [count and list, to distinguish from regressions]
```

The PM can verify any of these in under 60 seconds. If the structured fields match the prose description, confidence in the prose is higher. If they don't match, the prose is suspect.

### Debrief Review Checklist (for PM)

When reviewing a debrief, check:
- [ ] Do the test counts in the debrief match a recent `pytest` run?
- [ ] Does the "files modified" list match `git diff --stat` for the relevant commits?
- [ ] Are the commit hashes real? (`git log --oneline` to verify)
- [ ] Does the "concerns" section contain specific observations, or is it generic/empty?
- [ ] If the retrospective says "no issues," does the commit history show a clean single-pass, or multiple fix-up commits suggesting issues that weren't reported?

## When to Use

- When establishing a new multi-agent project and designing the debrief format
- When a debrief is discovered to be inaccurate (promotes awareness of the trust boundary)
- When deciding how much to trust a debrief's prose sections for planning purposes

## Anti-Patterns

- **Assuming all debrief content is equally reliable.** Machine-verified fields and prose retrospectives have fundamentally different integrity levels. Treat them accordingly.
- **Assuming all prose is unreliable.** Agents are generally honest reporters. The issue isn't deception — it's thoroughness under context pressure. An agent at 90% context utilization will write a worse retrospective than one at 50%, not because it's lying but because it's depleted.
- **Auditing every debrief.** The cost exceeds the benefit for routine WOs. Reserve cross-agent audit for high-stakes changes.
- **Not auditing any debriefs.** Without occasional verification, the trust assumption is untested. Spot-checks keep the system honest.

## Real Example

The proving-ground project's WO-RNG-PROTOCOL-001 debrief reported a mid-session course correction: `replace_all` of `RNGManager` accidentally hit constructor call sites in `replay_runner.py` and `session_log.py`. The debrief described the error and the fix. This claim is partially verifiable — the commit history would show whether a fix-up commit exists. But the agent's statement that it "caught the error immediately in the first test run" is prose-only — there's no artifact confirming the timing. The PM trusts this claim based on the overall consistency of the debrief, not because it's independently verified.
