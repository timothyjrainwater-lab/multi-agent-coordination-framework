# Pattern: Proactive Assumption Sweep Cadence

## Problem

The [Hidden Assumption Sweep](HIDDEN_ASSUMPTION_SWEEP.md) is a powerful triage tool — but it is reactive by design. You run it when a question "feels larger than it should." The problem: many hidden assumptions don't produce that feeling. They feel settled. They feel like known ground. They surface quietly as production failures weeks later.

The reactive trigger misses the class of assumptions that are confidently wrong.

**The failure pattern:**
- A subsystem is built across 5 batches
- Each WO is scoped correctly, tested correctly, and accepted
- Nobody runs an assumption sweep because nothing triggered one
- At batch 8, a builder questions whether a boundary condition was ever explicitly specified
- It wasn't — it was assumed from context by the first builder, inherited by all subsequent builders, and the assumption is wrong
- The correction requires touching 4 resolvers and 12 tests

**Root cause:** Hidden assumptions compound silently. The cost of catching them grows with the number of layers built on top of them. A reactive sweep catches the assumption at trigger time — but trigger time is often well after the assumption has already been load-bearing for multiple batches.

---

## Solution

### Rule

**Run a proactive Hidden Assumption Sweep on a fixed cadence — not just when triggered.**

Pair the sweep with your existing sweep audit cadence (every N batches, or at each batch boundary). The sweep is brief: 10 questions, 3-10 minutes, one artifact filed regardless of outcome. The goal is not to find problems — it is to confirm that what you think is settled actually is.

### Cadence

| Trigger | Sweep Type | Who Runs It |
|---------|-----------|-------------|
| Every 5 engine batches | Proactive subsystem sweep | PM, paired with the sweep audit |
| After any cross-cutting change | Full boundary sweep | PM |
| After a new spec or design document is added | Scope sweep | PM |
| When a builder WO is rejected for spec ambiguity | Targeted sweep on the ambiguous assumption | PM |
| When a test passes but behavior "feels wrong" | Targeted sweep on the assumption the test is validating | Auditor |

The proactive sweep pairs with the Sweep Audit Protocol — when you schedule a read-only subsystem audit, you also run the 10-question sweep on that subsystem's key assumptions before the audit is dispatched. This way, assumption gaps are caught before new code is written against them.

### The 10-Question Sweep (Quick Reference)

Answer each question YES / NO / UNKNOWN for the subsystem being reviewed.

1. **Is there a missing assumption here, or just a missing feature?** (Assumptions are more dangerous — they affect everything built on top of them)
2. **Does any open WO or milestone change meaning if this assumption is wrong?**
3. **Does this touch a system boundary?** (Between subsystems, between system and users, between two resolvers that hand off to each other)
4. **Does the system currently fail closed (silent, safe) or fail open (inventing answers) at this point?**
5. **Does it require a human judgment layer that the source documentation never explicitly defines?**
6. **Does it affect multiple subsystems, or just one?**
7. **Can a user or adversary expose this gap through normal, reasonable behavior?**
8. **Is there an observable failure mode, or is this a silent failure?**
9. **Who is authorized to make the ruling if the system cannot?**
10. **What state must the system track for any ruling here to be meaningful?** (If the required state is not tracked, flag as a world-model gap)

### Escalation Rule

**Escalate to STRATEGY (not WO) if:**
- 3 or more answers are UNKNOWN, **or**
- any answer changes the definition of done for anything in progress, **or**
- any boundary is found to be fail-open

When in doubt, escalate. Filing a strategy item that turns out to be minor costs nothing. Building 3 more batches on a wrong assumption costs everything.

---

## Output Format

Every sweep produces a short artifact filed immediately — not noted for later:

```markdown
**Sweep ID:** SWEEP-[SUBSYSTEM]-[NNN]
**Date:** [YYYY-MM-DD]
**Subsystem:** [subsystem name]
**Trigger:** [Cadence (every N batches) | Cross-cutting change | WO rejection | Manual]
**Conducted by:** [PM / Auditor]

**Question answers:**
1. YES / NO / UNKNOWN — [one sentence]
2. YES / NO / UNKNOWN — [one sentence]
... (all 10 questions)

**Unknown count:** N

**Classification:**
- [ ] No issues — all assumptions confirmed, proceed
- [ ] STRATEGY item filed — [one sentence describing the gap]
- [ ] WO filed — [one sentence describing the scoped fix]
- [ ] Deferred — [one sentence rationale]
```

File it in the PM inbox or the project's findings log. Use lifecycle `NEW` if it requires action, `ARCHIVE` if the result is clean.

---

## Subsystem Rotation

Run sweeps in the same rotation as your audit cadence. Don't sweep the same subsystem twice in a row. Maintain a sweep log:

| Batch | Subsystem | Sweep ID | Result |
|-------|-----------|----------|--------|
| 5 | Attack resolution | SWEEP-ATTACK-001 | Clean |
| 10 | Condition stack | SWEEP-CONDITIONS-001 | STRATEGY filed |
| 15 | Spellcasting | SWEEP-SPELLS-001 | Clean |
| 20 | Action economy | SWEEP-ACTION-001 | WO filed |

The rotation ensures every major subsystem is reviewed before 3 full cycles pass. Subsystems with prior strategy items get earlier re-sweeps.

---

## Integration with Other Patterns

| Pattern | Integration |
|---------|-------------|
| [Hidden Assumption Sweep](HIDDEN_ASSUMPTION_SWEEP.md) | This pattern sets the cadence; that pattern provides the sweep protocol |
| [Sweep Audit Protocol](SWEEP_AUDIT_PROTOCOL.md) | Pair the assumption sweep with every audit dispatch — run sweep first, then dispatch auditor |
| [Research-to-Build Pipeline](RESEARCH_TO_BUILD_PIPELINE.md) | Assumption sweeps at the Brick stage surface judgment gaps before builders are dispatched |
| [Authority Tagging](AUTHORITY_TAGGING.md) | Question 5 (human judgment layer) frequently surfaces POLICY assumptions that need explicit tagging |

---

## Real Example

At batch 10 of the AIDM engine build, a proactive assumption sweep on the condition stack revealed that the question "what happens when two conditions with conflicting movement restrictions apply simultaneously?" had never been answered. The spec described each condition individually but never specified resolution order for conflicts.

The system's current behavior was fail-open: it applied whichever condition was processed last, silently. No test caught this because no test combined conflicting conditions.

**Without the proactive sweep:** The behavior would have continued silently. The gap would have been discovered when a player encountered a prone + paralyzed + slowed combination and the movement calculation produced an impossible value.

**With the proactive sweep:** The ambiguity was filed as a STRATEGY item. The operator made an explicit ruling (more restrictive condition wins). A builder WO formalized the priority table. The fix was one WO. Catching it after four more batches would have required touching seven.

---

## Anti-Patterns

- **Only running sweeps when triggered.** Assumptions that don't feel shaky are the most dangerous ones. The reactive trigger misses confidently-wrong assumptions.
- **Running the sweep without filing the output.** The sweep is only valuable as an artifact. A sweep run in conversation and forgotten is worth nothing.
- **Filing a WO for a 3+ UNKNOWN result.** That is a strategy item. A WO scoped against an unresolved assumption will make the assumption permanent.
- **Skipping the sweep when the batch feels clean.** "Nothing triggered" is not evidence that assumptions are correct. It is evidence that no trigger fired.
- **Sweeping the same subsystem repeatedly and skipping others.** Rotate. The rotation rule exists because the most dangerous assumptions are in the subsystems you stopped worrying about.

---

## When to Use

- Every N batches (pair with the sweep audit cadence)
- After any cross-cutting change (shared utility modified, central dispatcher changed)
- After a new spec or design document is added to the project
- When a builder WO is rejected because the spec was ambiguous

## When NOT to Use

- Instead of the Hidden Assumption Sweep reactive trigger — this pattern adds scheduled sweeps; it does not replace triggered ones
- For clearly scoped, local feature work with no cross-system implications
- Mid-sprint when momentum matters more than coverage — schedule for the batch boundary
