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

## F-009: Constructor Replacement via Blanket Edit

**Category:** Impact Analysis (Schema Cascade variant)
**Severity:** MEDIUM — caught by test suite but required mid-session course correction

**What happened:** WO-RNG-PROTOCOL-001 required replacing the concrete class `RNGManager` with the new `RNGProvider` Protocol in type annotations across 19 files. The builder used `replace_all` to change `RNGManager` to `RNGProvider` across each file. In two files (`replay_runner.py` and `session_log.py`), the same class name appeared both as a type annotation AND as a constructor call: `rng = RNGManager(master_seed)`. The blanket replacement changed both to `RNGProvider(master_seed)`, which fails because Protocols cannot be instantiated.

**Root cause:** `replace_all` operates on string patterns without semantic awareness. It doesn't distinguish between "this name appears as a type annotation" and "this name appears as a constructor call." The WO listed 8 specific files but said "all other core resolvers" — the builder used a uniform approach without checking whether each file had constructor sites.

**Fix created:** Before using `replace_all` on a class name, grep for `ClassName(` (with parenthesis) to identify files that use the class as both a type and a constructor. Handle those files separately with targeted edits. The builder's debrief documented this as a fragility observation for future type-level refactors.

---

## F-010: Dead Validation Rule (Silent Pass)

**Category:** False Confidence
**Severity:** HIGH — validation pass reports all-clear when the validated field is structurally empty

**What happened:** Two compile-time validation rules (CT-006: contraindication enforcement) and one runtime rule (RV-007: contraindication consistency check) were designed to catch narration errors — e.g., a fire spell narrated with ice visual effects. However, the `contraindications` field on `AbilityPresentationEntry` is always an empty tuple `()` because the SemanticsStage never generates contraindication data.

**Root cause:** The validation rules were written before the data they validate was populated. Both rules iterate over `contraindications` entries and check for violations. When the tuple is empty, the loop executes zero times, and the rule reports PASS — not because the data is correct, but because there's nothing to check.

**Detection:** Discovered during a roadmap audit that cross-referenced research findings (RQ-002: contraindications always `()`) against the WO scope (WO-COMPILE-VALIDATE-001 includes CT-006). The auditor flagged that shipping validation rules for an unpopulated field creates false confidence — the PM reviewing validation results could mistake "0 violations" for "verified correct."

**Fix created:** Validation rules that check a field which may be structurally empty should either: (a) report SKIP (not PASS) when the field is empty, with a warning that the check is dormant; or (b) be scoped into the same WO that populates the field, so the rule and its data land together.

---

## F-011: Parallel Implementation Path Drift

**Category:** Cross-Path Consistency
**Severity:** HIGH — 21 independent calculations drifted across two code paths over 30+ work orders; discovered only by a systematic audit

**What happened:** A result calculation had a "single-item" resolver and a "batch" resolver. The batch resolver was added later and reproduced the calculation logic independently rather than delegating. Over 30+ work orders, every WO that improved the calculation targeted the single-item resolver. No dispatch ever mentioned the batch resolver. The batch resolver drifted silently. A sweep audit eventually found that 21 independent calculation components had diverged between the two paths. No individual WO caused the drift — the drift accumulated because no dispatch enumerated the second path.

**Root cause:** WO dispatches are scoped to fix a specific known gap. They treat the target function as the only implementation. No one was responsible for asking "where else does this same logic exist?" The PM didn't know about the parallel paths; the builders weren't asked to look.

**Fix created:** PARALLEL_IMPLEMENTATION_PARITY pattern. Every WO modifying a resolver function must identify all parallel code paths before work begins. Builder verifies parity across all paths before filing the debrief. Missing parity check = debrief reject. When PM doesn't know all parallel paths, the dispatch instructs the builder to identify them as their first task.

---

## F-012: Specification Authority Gap

**Category:** Domain Correctness
**Severity:** HIGH — specification deviation shipped as engine behavior; discovered only by a debrief reviewer reading the original specification directly

**What happened:** A WO was dispatched to implement a calculation rule. The rule had a widely-used community variant that differed from the written specification. The WO dispatch had no authority tag. The builder used the community variant — plausible, locally reasonable, tests passed. The deviation was discovered at debrief by a reviewer who read the original specification directly. The implementation was 1.5× where the specification said 2×. Correcting it required a builder WO and downstream effects.

**Root cause:** The WO spec had no obligation to declare what authority the implementation should follow. The builder had no signal that this was a contested point. The PM had no obligation to check. The ambiguity flowed through the entire process undetected because it was never made visible.

**Fix created:** AUTHORITY_TAGGING pattern. Every WO touching domain logic must declare its authority source: SPEC (with citation) or POLICY (with operator sign-off). No third type. Agent judgment, community convention, and "obviously this is how it works" are not valid authority types. An ambiguity catalog maintained by the PM tracks rules with known community variants.

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
| F-009 | Impact Analysis | Pre-edit grep for constructor sites |
| F-010 | False Confidence | SKIP vs PASS for empty-field validation |
| F-011 | Cross-Path Consistency | Parallel implementation parity check |
| F-012 | Domain Correctness | Authority tagging (SPEC/POLICY) |
