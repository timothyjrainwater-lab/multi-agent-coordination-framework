# Pattern: Consume-Site Verification

## Problem

An agent implements a feature. The data field is populated. The tests pass. The WO is accepted. The feature does nothing at runtime.

This is the write-only trap: a feature can appear fully implemented if the acceptance criteria only checks that the data was written. The runtime effect — the actual change in behavior that the feature was supposed to produce — is never verified.

The failure is invisible because:
- The data field exists and has the right value
- The chargen or configuration pipeline is correct
- The gate tests for the WO all pass
- The debrief is accurate about what was done

What none of these verify: **is this field ever read by a runtime resolver?**

## Solution

Every WO that writes a field, registers data, or configures a value must verify the full consumption chain before filing its debrief.

### The Four-Layer Chain

```
Layer 1 (Write):   Where is the field written?
                   file:function or file:line_range

Layer 2 (Read):    Where is the field read at runtime?
                   file:function or file:line_range

Layer 3 (Effect):  What observable behavior changes because of that read?
                   Describe the runtime difference between field present vs. absent.

Layer 4 (Test):    Which gate test proves the runtime effect fires?
                   test_file:test_function — not a setup test, a behavior test
```

All four layers must be answered in the WO debrief. If any layer is missing, the WO is incomplete.

### When the Consume Site Doesn't Exist

Sometimes a WO correctly identifies that no consume site exists. This is the `CONSUME_DEFERRED` case. The builder:

1. Explicitly states the field is write-only (no runtime consumer yet)
2. Files a tracking finding: `FINDING-[ID]-CONSUME-DEFERRED: field X written at Y, no consumer exists`
3. States the expected consume site (the resolver that should eventually read it)
4. Marks the mechanic as write-only in the coverage map

The PM records the defer and ensures the tracking finding is in the open backlog. A `CONSUME_DEFERRED` field is not forgotten — it is explicitly parked.

**Acceptance gate:** No WO is ACCEPTED if a field is write-only without `CONSUME_DEFERRED` explicitly declared and a finding logged.

## The Corrected Definition of "Implemented"

A mechanic is **not** implemented merely because:
- Code exists
- Tests pass
- Data is populated
- A WO is accepted

A mechanic is implemented only when it satisfies all six criteria:

1. **Source-cited** — RAW or HOUSE authority tagged with a specific reference
2. **Code written** — the logic exists in the codebase
3. **Consumed at runtime** — an observable effect exists; the field is read and changes behavior
4. **Parity-checked** — if parallel implementation paths exist, both paths are consistent
5. **Tested** — a gate test specifically proves the runtime effect (not just setup)
6. **Reflected in planning artifacts** — the coverage map row is updated

"Code + passing tests" satisfies criteria 2 and 5 only. The other four are independent gates.

## Implementation

### In the WO Dispatch

The PM includes a **Consumption Chain** section in every WO that writes a field:

```markdown
## Consumption Chain

| Layer | Location | Description |
|-------|----------|-------------|
| Write | builder.py:142 | `ef[EF.MONK_UNARMED_DICE] = unarmed_dice` |
| Read  | attack_resolver.py:87 | `dice = ef.get(EF.MONK_UNARMED_DICE, "1d3")` |
| Effect | Monk unarmed strike uses correct dice per level instead of base 1d3 |
| Test  | test_monk_unarmed_gate.py::test_monk_level_4_uses_1d6 |
```

If the PM doesn't know the consume site at dispatch time, the dispatch instructs the builder to identify it as their first task.

### In the Builder Debrief

The builder confirms the consumption chain end-to-end:

```markdown
## Consume-Site Confirmation

- Write site: builder.py:142
- Read site: attack_resolver.py:87
- Observable effect: level 4 monk uses 1d6, not 1d3 (confirmed via test)
- Proof test: test_monk_unarmed_gate.py::test_monk_level_4_uses_1d6 — PASS
```

If the consume site was `CONSUME_DEFERRED`, the builder states this explicitly and logs the finding.

### PM Acceptance Check

Before issuing ACCEPTED, the PM verifies:
- Consume-site confirmation section exists in debrief
- Layer 2 (read site) is not empty and not `CONSUME_DEFERRED` without a logged finding
- Layer 4 (proof test) is a behavior test, not a setup test

## Anti-Patterns

**The setup test that passes:**
A test that verifies `ef[EF.SOME_FIELD] == expected_value` is a write verification, not a consume verification. It confirms the field was written correctly. It proves nothing about runtime behavior. A consume-site proof test must verify that the runtime output changes when the field is present vs. absent (or present with different values).

**The "eventually wired" justification:**
"We'll wire the consume site in the next WO." This is acceptable only if formally declared as `CONSUME_DEFERRED` with a logged finding. Undeclared write-only delivery treated as complete is the failure mode this pattern prevents.

**Counting data registration as implementation:**
A spell registry entry, a feat definition in a lookup table, or a creature stat block in a JSON file is not an implemented mechanic. It is a data dependency for an implementation. The consume site — the resolver that reads from the registry and changes combat behavior — is the implementation.

## Connection to Other Patterns

- **PARALLEL_IMPLEMENTATION_PARITY** — the parity check is Layer 4 of consume-site verification applied to multiple paths. Both patterns are required for mechanics with parallel resolver paths.
- **SWEEP_AUDIT_PROTOCOL** — sweeps catch write-only fields that slipped through individual WO acceptance. A consume-site sweep is a high-value audit pass.
- **RESEARCH_TO_BUILD_PIPELINE** — data ingested from research (OSS sources, SRD data) must also pass consume-site verification. Data existing in a file is not a runtime mechanic.
