# Metrics

Quantitative results from the D&D 3.5e referee engine project — the proving ground for this framework.

**Last Updated:** 2026-02-27

---

## Verification Results

| Domain | Formulas | Correct | Wrong | Ambiguous | Uncited |
|--------|----------|---------|-------|-----------|---------|
| A: Attack Resolution | 57 | 43 | 4→2 | 2→4 | 9 |
| B: Combat Maneuvers | 45 | 34 | 5 | 5 | 1 |
| C: Spells & Saves | 21 | 15 | 3→1 | 1→3 | 2 |
| D: Conditions | 38 | 28 | 4 | 4 | 2 |
| E: Movement & Terrain | 42 | 36 | 3 | 2 | 1 |
| F: Character Progression | 77 | 65 | 5→3 | 3→5 | 4 |
| G: Play Loop | 22 | 16 | 2 | 2 | 2 |
| H: Targeting | 18 | 16 | 1 | 1 | 0 |
| I: Geometry & Feats | 49 | 38 | 3 | 4 | 4 |
| **Total** | **338** | **255** | **30→~22** | **28** | **25** |

*Arrows indicate changes from research cross-referencing (Domain A re-verified, others estimated).*

**First-pass accuracy:** 75.4% (CORRECT / total)
**With AMBIGUOUS as non-bugs:** 83.7% ((CORRECT + AMBIGUOUS) / total)
**Estimated genuine bugs after cross-ref:** ~22 of 338 = 93.5% accuracy

---

## Bug Classification

30 bugs categorized into 8 error patterns:

| Pattern | Count | Example |
|---------|-------|---------|
| Missing modifier/multiplier | 6 | STR grip multiplier not applied |
| Wrong threshold/floor | 4 | min damage 0 instead of 1 |
| Condition not differentiated | 4 | Prone AC flat instead of melee/ranged |
| Missing field/parameter | 3 | Concentration DC missing spell level |
| Inverted condition | 2 | Soft cover applied to melee instead of ranged |
| Incorrect die/formula | 3 | Water fall d6 instead of d3, sunder hardcoded 1d8 |
| Incomplete enumeration | 2 | SIZE_ORDER missing 3 categories, Colossal footprint |
| Design decision misidentified | 6 | Cover values flagged as bugs, were intentional |

---

## Agent Coordination Metrics

| Metric | Value |
|--------|-------|
| Total agent sessions | 100+ |
| Parallel agent groups (max) | 7 simultaneous |
| Silent agent failure rate | 3/7 (43%) in one parallel dispatch |
| WOs requiring reclassification | 1 of 13 (WO-FIX-11, code structure mismatch) |
| Schema cascade underestimation | 1 of 13 (WO-FIX-03, 3 files scoped → 6 touched) |
| Cross-file consistency failures | 2 (Domain C verification, Domain A checklist) |
| Research cross-ref reclassifications | 4 confirmed (Domain A), ~8-10 estimated (all domains) |
| Total WOs dispatched (all types) | 100+ (fix, feature, research, governance, audit, framework) |
| Builder debriefs archived | 50+ |
| Research documents produced | 30 |
| Delivery batches completed | 25+ |
| Gate test suite size | 8,521+ tests |

---

## Fix Execution Metrics

| Metric | Value |
|--------|-------|
| Fix WOs dispatched | 13 (12 active, 1 retired) — Phase 1 only |
| Fix WOs completed | 11 of 12 |
| Fix WOs needing reclassification | 1 (WO-FIX-11) |
| Fix WOs partially completed | 1 (WO-FIX-12, BUG-F2/F3 unverified) |
| Tests passing after all Phase 1 fixes | 5,277 |
| Tests passing after all Phase 2 batches | 8,521+ |
| Tests updated (old wrong behavior) | 6 |
| Gold master files regenerated | 4 |
| Total commits for fix session | 9 |

---

## H1 WO Batch Metrics

| Metric | Value |
|--------|-------|
| H1 WOs completed | 7 |
| Tests passing after H1 batch | 5,804+ |
| Builder commit failures recovered (one batch) | 4 (3/7 agents silently failed to commit) |
| Integration Constraint Policy | Codified — no new infrastructure WOs until canary runs |
| Integration break points found by canary | 4 (all invisible to unit tests) |

---

## Phase 2 Batch Delivery Metrics

| Metric | Value |
|--------|-------|
| Batches completed (Phase 2) | 20+ (Batches I through R+) |
| Average gate tests per batch | 8 (range: 6-11) |
| Average accepted WOs per batch | 4 |
| Regression failures introduced | 0 (zero regressions across all accepted batches) |
| Ghost WOs dispatched (feature already implemented) | ~3 (identified via pre-dispatch verification) |
| Parallel path drift incidents caught | 1 (F-011, 21-modifier divergence, discovered by sweep audit) |
| Regression spiral incidents | 1 (F-ML-004, agent burned context on pre-existing failures) |
| PM commits sweeping staged builder code | 1 (F-ML-005, caught post-hoc; audit trail corrected) |
| Spec authority gap incidents | 1 (F-012, community variant shipped over specification value) |

---

## Enforcement Tier Effectiveness

| Tier | Stickiness | Example |
|------|-----------|---------|
| Tier 1: Test-enforced | ~100% | Boundary law tests, WorldState immutability |
| Tier 2: Process-enforced | 70-80% | Dispatch templates, handoff checklists |
| Tier 3: Prose-enforced | 40-60% | PM inbox 10-item cap (violated 2.3x) |

---

## Context Window Observations

| Observation | Data Point |
|-------------|-----------|
| Largest agent token consumption | ~170K tokens (WO-FIX-01/02 attack resolvers) |
| Typical WO agent consumption | 30K-80K tokens |
| PM summary effective length | 7 items / ~500 tokens |
| Full debrief length | ~2,000-4,000 tokens |
| Compression ratio (full → summary) | 4:1 to 8:1 |
