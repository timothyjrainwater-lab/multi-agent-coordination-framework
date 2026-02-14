# Failure Catalog

Real coordination failures encountered during the D&D 3.5e referee engine build. Each entry documents what happened, why, and what protocol was created to prevent recurrence.

---

## F-001: Partial Update Drift

**Category:** Cross-File Consistency
**Severity:** HIGH — caused downstream agents to dispatch fix WOs for non-bugs

**What happened:** Domain A re-verification reclassified 4 bugs from WRONG to AMBIGUOUS. The agent updated the checklist and WRONG_VERDICTS_MASTER but missed DOMAIN_C_VERIFICATION.md. The verification file still said "3 WRONG / 1 AMBIGUOUS" while the checklist said "1 WRONG / 3 AMBIGUOUS."

**Root cause:** No protocol required updating all files containing a fact in the same commit. The agent updated the files it had in context and moved on.

**Fix created:** Cross-File Consistency Gate pattern. All-or-nothing updates. If a fact changes, every file referencing it must be updated in the same commit.

---

## F-002: Design Decision Blindness

**Category:** Verification Pitfall
**Severity:** HIGH — 4 bugs reclassified in one domain alone, estimated 8-10 across all domains

**What happened:** The initial verification pass compared code against raw SRD text without checking the research corpus. Cover values that were intentionally different (documented in RQ-BOX-001 Finding 3 as a design decision) were flagged as WRONG.

**Root cause:** The verification dispatch didn't include a required reading list for research documents. The verifier had no reason to suspect the code diverged intentionally.

**Fix created:** Research cross-reference requirement added to all verification and fix WO dispatches. Agents must search research documents for the relevant mechanic before writing WRONG verdicts.

---

## F-003: Silent Agent Completion

**Category:** Agent Reliability
**Severity:** HIGH — 43% of parallel agents (3/7) reported completion with zero code changes

**What happened:** Seven background agents were dispatched to execute fix WOs in parallel. Three agents reported task completion but wrote zero changes to disk. The coordinator discovered this during the commit review phase when `git diff` showed no changes for the affected files.

**Root cause:** Unknown — possibly agents hit edit tool failures and reported success based on having "processed" the task rather than having written changes. The agents' internal state said "done" but the external artifact (modified files) didn't exist.

**Fix created:** Post-completion verification gate: after any agent reports completion, `git diff` the target files before accepting the result. An agent saying "done" is not the artifact — the diff is.

---

## F-004: Stale Document Cascade

**Category:** Machine Truth vs. Prose Truth
**Severity:** MEDIUM — caused incorrect confidence assessments

**What happened:** PROJECT_KNOWLEDGE_SYNTHESIS.md (dated 2026-02-08) claimed 0.98 confidence for the CP-09-17 foundation. The bone-layer verification (completed 2026-02-14) found 30 formula-level bugs in that foundation. An agent reading the synthesis document would have false confidence in code that had a ~5.5% error rate.

**Root cause:** No process required updating analysis documents after new data invalidated their conclusions. The synthesis was written once and never refreshed.

**Fix created:** Session Bootstrap pattern — agents run machine-verified commands before trusting prose documents. Documents that make quantitative claims should be dated and carry a staleness warning.

---

## F-005: WO Target Mismatch

**Category:** Dispatch Accuracy
**Severity:** MEDIUM — one WO (WO-FIX-11) targeted code that didn't exist as described

**What happened:** WO-FIX-11 described an "action cost table at play_loop.py lines 71-86" with trip/disarm/grapple classified as "standard." The actual code routes combat maneuvers via isinstance checks on Intent objects, not a string-to-cost lookup table. The WO was correct about the SRD rule but wrong about the code structure.

**Root cause:** The WO was written from verification findings (which identified the wrong classification) but the fix description assumed a code structure that didn't match reality. The verifier analyzed behavior, not implementation.

**Fix created:** Fix WOs should include "read the code at this location and verify the structure matches this description" as a pre-condition. If the structure doesn't match, report back before attempting the fix.

---

## F-006: Gold Master Surprise

**Category:** Test Infrastructure
**Severity:** LOW — caused transient confusion but no lasting damage

**What happened:** After fixing attack damage calculations (WO-FIX-01/02), four gold master test fixture files changed. The agents regenerated them, which was correct — the old gold masters encoded wrong damage values. But the diff was large (584 lines in one file) and initially alarming.

**Root cause:** Gold master tests are snapshot-based. Any change to the code they exercise changes their output. This is expected but not documented as a consequence of damage formula fixes.

**Fix created:** Fix WOs that touch core resolvers should note "gold master regeneration expected" as a predicted side effect. This prevents the next reviewer from treating the diff as suspicious.

---

## F-007: Context Window Exhaustion Mid-Task

**Category:** Resource Management
**Severity:** MEDIUM — causes incomplete work and forced handoffs

**What happened:** Multiple agents working on complex WOs (particularly WO-FIX-01/02, the attack resolver rewrite) consumed large amounts of context reading files, making edits, running tests, and debugging failures. The attack resolver agent used 170K+ tokens before completing.

**Root cause:** Complex tasks in large files consume context rapidly. The attack resolver files are 1,000+ lines each. Reading, editing, re-reading, and test-debugging each consumes context that doesn't recover.

**Fix created:** WO sizing guideline — if a WO requires reading more than 500 lines of code and making changes across multiple functions, consider splitting it into sub-WOs. The coordinator should monitor agent token consumption and be prepared to split tasks that exceed 60% of context budget during the reading phase.

---

## F-008: Schema Cascade Underestimation

**Category:** Impact Analysis
**Severity:** MEDIUM — WO took 3x longer than estimated due to unlisted file dependencies

**What happened:** WO-FIX-03 (adding ac_modifier_melee/ranged to conditions schema) was scoped to touch 3 files: conditions.py, attack_resolver.py, full_attack_resolver.py. In reality, the fix also required changes to:
- `conditions.py` serialization (to_dict/from_dict)
- `aidm/core/conditions.py` aggregation (get_condition_modifiers)
- 4 test files with assertions testing the old (wrong) behavior

The WO listed 3 files. The fix touched 6.

**Root cause:** Schema changes propagate through serialization, aggregation, and tests. The WO author traced the direct consumers but not the infrastructure layers.

**Fix created:** Schema-change WOs should trace the full propagation path: definition → serialization → aggregation → consumers → tests. A checklist: "Did you update to_dict? from_dict? Any aggregation function? Any test asserting the old value?"

---

## Summary

| ID | Category | Fix Pattern |
|----|----------|-------------|
| F-001 | Cross-File Consistency | All-or-nothing commits |
| F-002 | Verification Pitfall | Research cross-reference requirement |
| F-003 | Agent Reliability | Post-completion git diff gate |
| F-004 | Machine Truth vs. Prose | Session bootstrap, staleness warnings |
| F-005 | Dispatch Accuracy | Pre-condition verification step |
| F-006 | Test Infrastructure | Gold master regeneration prediction |
| F-007 | Resource Management | WO sizing guidelines |
| F-008 | Impact Analysis | Schema cascade checklist |
