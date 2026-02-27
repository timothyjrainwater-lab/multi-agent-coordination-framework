# Template: Audit Work Order Dispatch

An audit WO is fundamentally different from a builder WO. The auditor reads code, answers specific questions, and files findings. The auditor **never writes production code**. This template enforces that separation.

See: [Sweep Audit Protocol](../patterns/SWEEP_AUDIT_PROTOCOL.md)

---

```markdown
# [AUDIT-WO-ID]: [Subsystem Name] Audit — Batch [N]

**Assigned to:** [Auditor agent — e.g., "Auditor (Sonnet)" or "Auditor (Opus) for cross-cutting"]
**Date:** [YYYY-MM-DD]
**Priority:** [HIGH | MEDIUM | LOW]
**Status:** [DRAFT | DISPATCHED | IN PROGRESS | COMPLETE]
**Trigger:** [Cadence (every N batches) | Cross-cutting change | Manual escalation]

---

## READ-ONLY MANDATE

This is a read-only audit. You may not modify production code.

If you find a critical flaw:
1. Flag it with SEVERITY: CRITICAL in the findings table
2. Note it prominently at the top of your debrief
3. The PM will triage into a corrective builder WO

Do not attempt to fix what you find. The audit value is the independent read. Once you write code, the audit is over.

---

## Audit Scope

**Primary target:** [file or module — full read]
**Secondary targets:** [related files — targeted read of specific sections only]
**Out of scope:** [adjacent subsystems NOT covered by this audit]

**Files to read:**

- `path/to/primary_module.py` (full read)
- `path/to/secondary_module.py` (lines XX-YY, section: [description])
- `path/to/spec_or_design_doc.md` (for specification cross-reference)

---

## Questions the Auditor Must Answer

For each question: cite the code location (file:line), state YES/NO/PARTIAL, and explain your answer in 1-3 sentences.

1. **[Correctness question]**
   *e.g., "Is the [rule name] calculation at [file:line] consistent with [specification section]?"*

2. **[Parallel path question]**
   *e.g., "Does the [batch path] at [file:line] produce the same result as the [single-item path] at [file:line] for the same inputs?"*

3. **[Enforcement question]**
   *e.g., "Is [guard condition] actually blocking what it should, or does it have gaps in [edge case]?"*

4. **[Coverage question]**
   *e.g., "Are all expected cases handled in [function], or are there unhandled edge cases?"*

5. **[Consistency question]**
   *e.g., "Do [subsystem A] and [subsystem B] use consistent definitions of [concept]?"*

Add questions as needed. Each question should be answerable YES/NO/PARTIAL from the code — avoid open-ended research questions.

---

## Finding IDs

Use IDs in the format: `FINDING-AUDIT-[SUBSYSTEM]-[NNN]`

Example: `FINDING-AUDIT-CONDITIONS-001`

Severity levels:
- **CRITICAL** — System behavior is currently incorrect in a way that affects all users of this subsystem. PM halts other work, drafts corrective WO immediately.
- **HIGH** — Significant gap with broad or visible impact. Queue for next dispatch, high priority.
- **MEDIUM** — Gap with limited scope or workaround available. Queue within 2 batches.
- **LOW** — Minor inconsistency or style issue. Queue when convenient.
- **INFO** — Architecture observation, no action required. Log for reference.

---

## Debrief Format (3-pass)

**Pass 1 — Question Answers:**
Answer each dispatch question with file:line citations. Include a findings table.

Findings table format:
| ID | Severity | Status | Summary |
|----|----------|--------|---------|
| FINDING-AUDIT-[SUBSYSTEM]-001 | HIGH | OPEN | [One sentence] |

**Pass 2 — PM Summary (≤100 words):**
Distill the key signal. What is the state of this subsystem? What are the highest-priority findings? What should the PM do first?

**Pass 3 — Retrospective:**
Patterns noticed. Architectural observations. Recommendations for builder WOs. Were there questions you couldn't answer from the files in scope? Note what additional reading would resolve them.

---

## What NOT to Do

- Do NOT modify any production files
- Do NOT draft builder WO dispatches — the PM triages findings into WOs
- Do NOT leave findings in session memory — file every finding before closing
- Do NOT answer questions with "probably" or "seems like" — read the code and state what it does
- Do NOT treat test files as ground truth for behavior — read the implementation
```

---

## Usage Notes

- **Scope is critical.** An audit of "everything" fails — the scope is too broad for a single context window. One primary file (full read) + 2-4 secondary files (targeted reads) + a specific question list is the right size.
- **Questions drive the value.** Vague questions produce vague findings. Each question should be answerable YES/NO/PARTIAL from code inspection.
- **CRITICAL findings stop the batch.** If the auditor flags CRITICAL, the PM should triage before dispatching the next builder batch. Don't run builders in parallel with an unresolved CRITICAL finding in the same subsystem.
- **Rotate subsystems.** Don't audit the same subsystem twice in a row. Maintain an audit coverage log — see SWEEP_AUDIT_PROTOCOL for the recommended rotation model.
- **Auditor ≠ builder.** If the auditor writes production code, they're no longer auditing. If you find yourself writing a fix, stop, file it as a finding, and wait for a builder WO.

## When to Use

- Every N delivery batches (see cadence table in SWEEP_AUDIT_PROTOCOL)
- After any cross-cutting change (shared utility modified, schema field added, central dispatcher changed)
- When related subsystems "feel inconsistent" but you can't pin the gap
- Before a major milestone or external demonstration

## When NOT to Use

- Instead of a builder WO for a known gap — audits find unknowns; builders fix knowns
- Mid-sprint when you need delivery momentum — schedule the audit for the batch boundary
- For the first 2-3 batches of a new project (not enough code to audit)
