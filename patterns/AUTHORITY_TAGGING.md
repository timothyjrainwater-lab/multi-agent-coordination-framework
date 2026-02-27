# Pattern: Authority Tagging

## Problem

When a builder implements domain logic, they make choices. Some of those choices are obvious from the specification. Others are ambiguous. A few are actively disputed in the community — commonly-understood one way, but technically specified another.

Without explicit authority tagging, builders resolve ambiguities by:
1. Using the most common community interpretation
2. Making a locally reasonable judgment
3. Reproducing what similar systems do

All three produce the same outcome: **behavior that is plausible but unverifiable**. When a reviewer or auditor later asks "why does this work this way?" there's no answer traceable to a source. The behavior is a guess that passed tests.

**The specific failure mode:** A builder implements a rule using a widely-known community variant that differs from the actual written specification. The implementation is "correct" in the sense that it matches what most practitioners expect — but incorrect against the authoritative source. The deviation ships, takes effect, and is only discovered when someone reads the original specification directly. At that point, correcting it is a breaking change with downstream effects.

**The root cause:** The WO spec didn't declare what authority the implementation should follow. The builder had no signal that this was a contested or ambiguous point. The PM had no obligation to check. The ambiguity flowed through the entire process undetected because it was never made visible.

## Solution

### Rule

**Every WO that implements domain logic must declare its authority source before dispatch.**

Two valid authority types:

| Type | Definition | Required documentation |
|------|-----------|----------------------|
| **SPEC** | Rules As Written from the authoritative specification | Citation: [document] [section/page] |
| **POLICY** | Explicit, versioned, operator-approved decision | Rationale + operator sign-off logged |

**No third type.** Agent judgment, community convention, and "obviously this is how it works" are not valid authority types. If neither SPEC nor POLICY covers a case, the correct action is to refuse, log the gap, and escalate — never to guess.

### The Authority Tag in WO Dispatches

Every WO targeting domain logic includes:

```markdown
## Authority Tag

**Rule:** [name of the rule or behavior being implemented]
**Authority type:** SPEC | POLICY
**Source:** [document and location if SPEC; policy ID and rationale if POLICY]
**Disputed:** Yes/No — [if Yes, note the common variant and explain why spec takes precedence]
```

Example:
```markdown
## Authority Tag

**Rule:** Two-handed weapon attack multiplier
**Authority type:** SPEC
**Source:** [Specification document] Section 3.2, "Two-Handed Weapons"
**Disputed:** Yes — common community convention uses 1.5× multiplier; specification explicitly states 2×. Implement per specification.
```

When the PM doesn't know whether a rule is disputed, the check is: *"Is there any reason someone might implement this differently?"* If yes, the authority tag is mandatory.

### The Ambiguity Catalog

Maintain a running list of rules that have documented ambiguities, community variants, or specification gaps. Before dispatching any WO touching domain logic, the PM checks this catalog.

```markdown
| Rule | Ambiguity | Specification says | Common variant |
|------|-----------|-------------------|----------------|
| [Rule name] | [Description of the ambiguity] | [What the spec actually says] | [What people often do instead] |
```

This catalog is a PM-maintained artifact. Builders never need to see it — the WO spec should already have the right authority tag. But the PM needs it to know which rules require extra scrutiny.

### When There Is No Specification

Some rules are genuinely unspecified. The specification doesn't cover the case. This is a **policy gap**, not a specification choice.

Correct handling:
1. The WO includes an `UNSPECIFIED` flag in the authority tag
2. The PM escalates to the operator for a policy decision before dispatch
3. The operator makes an explicit policy choice (even "default to the most common interpretation" is a policy)
4. The policy is documented and versioned
5. The WO ships with `POLICY` authority, citing the operator decision

**Never resolve an unspecified case by letting the builder guess.** The guess will be locally reasonable and permanently wrong in an undiscoverable way.

### Detecting Contested Rules

Some signals that a rule is contested:
- Community forums have threads arguing about it
- Multiple implementations of similar systems handle it differently
- The specification text is ambiguous or has errata
- The rule involves stacking, multipliers, or interaction between two systems (these are disproportionately contested)
- The rule was changed between versions of the specification

When any of these signals are present, treat the rule as contested and require an explicit authority tag.

## Implementation

### Adding Authority Tags Retroactively

For existing implementations without authority tags:

1. Auditor identifies the behavior
2. PM locates the specification citation
3. PM verifies the implementation matches the specification
4. If it matches: add a retroactive tag in a comment or governance doc
5. If it doesn't match: file a corrective WO

This is discovery work, not rework — don't fix what doesn't need fixing. The audit is looking for deviations, not just confirming the existing tags.

### The PM's Pre-Dispatch Checklist Addition

Add to the PM's WO dispatch checklist:

```
[ ] Does this WO touch domain logic?
      If yes:
      [ ] Is the rule's specification unambiguous?
      [ ] Does the rule appear in the ambiguity catalog?
      [ ] Is the authority tag complete in the WO?
      [ ] If contested: does the WO explicitly call out the variant and confirm spec takes precedence?
```

### Trust Hierarchy (For Specification-Rich Domains)

In domains with layered specifications (primary spec, errata, designer intent, community rulings), the PM maintains an explicit trust hierarchy:

1. Primary specification (highest authority)
2. Official errata to the primary specification
3. Designer clarifications (where documented)
4. Conservative interpretation of ambiguous cases
5. Community consensus (lowest authority, only when all above are silent)

The hierarchy is project-specific. The point is that it exists and is written down, so every WO can cite a level, not just a vague "this is how it works."

## When to Use

- Any project implementing rules from an external specification (a protocol, a standard, a rulebook, a regulation)
- Any domain where community practice diverges from written specification
- Any project where different implementations of the same system exist and disagree

## When NOT to Use

- Pure algorithmic work with no domain authority questions (e.g., "sort these items by date")
- Projects where the operator is also the specification authority (they can't contradict themselves)
- UI/UX changes with no correctness criteria

## Real Example

A project was implementing a calculation rule. The rule had a widely-used community variant that differed from the written specification. The WO dispatch had no authority tag — it simply said "implement [rule]."

The builder used the community variant. The implementation shipped. Tests passed.

Six batches later, a debrief reviewer read the original specification and noticed the discrepancy. The implementation was 1.5× where the specification said 2×. Correcting it required a builder WO, a gate test update, and a governance patch to document the correction.

**If the WO had included an authority tag** citing the specification and flagging the common variant, the builder would have seen the flag, implemented per specification, and the error would never have shipped.

**Fix:** Added mandatory authority tagging to all WOs touching rules with known community variants. Maintained an ambiguity catalog. The PM checks the catalog before every dispatch targeting rule implementations.
