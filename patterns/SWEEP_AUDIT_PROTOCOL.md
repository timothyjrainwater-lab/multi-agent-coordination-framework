# Pattern: Sweep Audit Protocol

## Problem

Iterative development with many small work orders has a structural blind spot: **each WO fixes the problem it was written for, but no single WO sees the system as a whole.**

A builder executing WO-47 fixes one calculation. A builder executing WO-51 fixes another. Each debrief is correct. Each gate test passes. But no one has looked at whether the calculation in WO-47 and the calculation in WO-51 use consistent logic across the same code path. No one has looked at whether 30 sequential WOs have drifted from the original specification. No one has looked at whether the sum of correct individual fixes has produced a coherent whole.

**The damage accumulates invisibly.** Individual WOs can all be correct while the system-as-a-whole has diverged from what it's supposed to do. This is the multi-WO coherence problem.

Without periodic audits:
- Parallel code paths drift apart silently (see PARALLEL_IMPLEMENTATION_PARITY)
- Modifier calculations accumulate inconsistencies across subsystems
- Sequential WOs make locally-correct choices that are globally incoherent
- The gap between "what we've built" and "what the spec says" grows with every batch

The root cause: no agent in the continuous development loop has the job of looking at the complete picture. Builders have narrow scope. PMs have high-level state. Nobody does periodic comprehensive subsystem reads.

## Solution

### Rule

**After every N delivery batches, file one read-only audit WO targeting a specific subsystem.**

The auditor reads code, files findings, and never writes production code. The PM triages findings into builder WOs. The sweep cycle catches what sequential narrow WOs cannot.

### The Audit Cadence

The cadence is configurable based on project velocity, but a workable default:

| Project Phase | Cadence | Trigger |
|--------------|---------|---------|
| Early (< 20 batches) | Every 10 batches | Time-based cadence |
| Mid (20-50 batches) | Every 5 batches | Batches completed |
| Late (50+ batches) | Every 3 batches | Batches completed OR a cross-cutting change |

**Cross-cutting trigger:** Any WO that touches a shared utility, central dispatcher, or schema definition should trigger an immediate audit of the subsystems that depend on it — regardless of cadence.

### Subsystem Rotation

Audits should rotate across subsystems systematically. Don't audit the same subsystem twice in a row. Suggested rotation model:

1. Attack/calculation paths
2. State management and persistence
3. Action economy and turn sequencing
4. Condition and duration tracking
5. Spellcasting / special ability resolution
6. Output and narration layer
7. Chargen / entity initialization
8. ...then repeat

This prevents blind spots from forming in under-reviewed subsystems.

### The Auditor Role

The auditor is a distinct role from the builder. Key constraints:

| Auditor Authorities | Auditor Boundaries |
|--------------------|-------------------|
| Read all files in scope | Does not modify production code |
| File FINDING-AUDIT-* entries | Does not draft builder WOs (PM does) |
| Flag critical issues immediately | Does not fix what it finds |
| Read cross-subsystem for coherence | Does not make architecture decisions |

**The auditor's output is findings, not fixes.** This separation is important. An auditor who also fixes is no longer doing an audit — they're doing a builder session with extra reading. The audit value is the independent read. Once the auditor starts writing code, the independent read is over.

### Audit Dispatch Requirements

An audit WO dispatch must include:

```markdown
## Audit Scope
Primary target: [file or module]
Secondary targets: [related files]
Files to read: [list]

## Questions the Auditor Must Answer
For each question, cite the code location (file:line) and the specification reference:
1. [Specific yes/no question about implementation correctness]
2. [Specific yes/no question about parallel path consistency]
3. [Specific yes/no question about enforcement — is this rule actually blocking what it should?]
...

## Finding IDs
Use IDs: FINDING-AUDIT-[SUBSYSTEM]-NNN

## Read-Only Mandate
This is a read-only audit. Do not fix anything. File findings. PM will triage into builder WOs.
If you find a critical flaw, flag it with SEVERITY: CRITICAL and notify immediately.
```

### Audit Debrief Format

Three-pass format:

**Pass 1:** Per-question answers with file:line citations. Findings table.
**Pass 2:** PM summary ≤100 words. Signal-to-noise distilled.
**Pass 3:** Retrospective — patterns noticed, kernel/architectural observations, recommendations for builder WOs.

**Findings table columns:** ID, Severity (CRITICAL/HIGH/MEDIUM/LOW/INFO), Status (OPEN/CLOSED), Summary

### Severity Escalation

| Severity | Definition | PM Action |
|----------|-----------|-----------|
| CRITICAL | System behavior is currently incorrect in a way that affects all users of this subsystem | Halt other work, draft corrective WO immediately |
| HIGH | Significant gap with broad or visible impact | Queue for next dispatch, high priority |
| MEDIUM | Gap with limited scope or workaround available | Queue within 2 batches |
| LOW | Minor inconsistency or style issue | Queue when convenient |
| INFO | Architecture observation, no action required | Log for future reference |

## Implementation

### Choosing Audit Scope

Don't audit "everything" — that's too broad for a single context window. Choose:
- One primary file or module (full read)
- 2-4 secondary files (targeted read of relevant sections)
- A specific set of questions (see template)

**A 500-line primary target with 8 specific questions** produces a better audit than "read the whole subsystem" with no questions.

### The Audit WO vs. the Builder WO

These are fundamentally different dispatches:

| Dimension | Builder WO | Audit WO |
|-----------|-----------|----------|
| Output | Code changes + debrief | Findings memo only |
| Files modified | Production code | None |
| Goal | Implement a specific thing | Assess whether the right things are implemented |
| Scope | Narrow (2-5 files typical) | Can be broader (primary + secondary targets) |
| Agent guidance | "Here is what to build" | "Here are the questions to answer" |

### Tracking Audit Coverage

Keep a record of what has been audited:

```markdown
| Subsystem | Last Audited | Findings | Status |
|-----------|-------------|----------|--------|
| Attack calculation | Batch 20 | 3 MEDIUM, 2 LOW | All resolved |
| Condition tracking | Batch 18 | 1 HIGH, 1 LOW | HIGH resolved, LOW queued |
| State management | Never | — | Due |
```

This table prevents the same well-understood subsystem from being audited repeatedly while dark corners accumulate debt.

## When to Use

- Any project that has completed 5+ delivery batches
- Any time a cross-cutting change lands (new shared utility, schema refactor, new field)
- Any time you notice that related subsystems "feel inconsistent" but can't pin the gap
- Before a major version, milestone, or external demonstration

## When NOT to Use

- First 2-3 batches of a project (not enough code to audit)
- Mid-sprint, when you need momentum — schedule the audit for the batch boundary
- Instead of a builder WO for a known gap — audits find unknowns; builders fix knowns

## Real Example

A project had completed 25 delivery batches. Each batch was individually correct. A sweep audit of the calculation subsystem found 21 independent modifier values that had drifted across two parallel code paths. No individual WO had caused the drift — the drift accumulated as sequential WOs targeted one path without knowing about the other.

The audit also found that an intermediate result was being computed correctly in the "single item" path but incorrectly in the "batch" path, and that one modifier had been wired with the wrong sign in 3 of 8 places it was applied.

None of these findings were visible in the test suite, because the test suite tested each path separately. The audit caught what sequential unit tests structurally cannot catch: the coherence of the whole.

**Result:** 3 corrective builder WOs were dispatched. The parallel path architecture was refactored to eliminate the structural source of drift.
