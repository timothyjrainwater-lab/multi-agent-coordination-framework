# Integration Canary

## Problem

Each agent builds and tests its piece in isolation. All unit tests pass. But nobody tries to use the product end-to-end. Constraints (boundary laws, frozen schemas, protocol interfaces) produce consistency within modules but cannot produce integration across modules.

**Symptoms:**
- "Works in isolation, fails in integration"
- Data bridges wired but dormant (code exists, data never flows)
- Validation rules that structurally cannot fire (the field they check is always empty)
- Compile stages that exist but aren't registered in production callers
- 5,000+ unit tests passing, zero integration tests

## Solution

Before dispatching new infrastructure WOs, run one script that exercises the full product path. Whatever breaks is your next WO.

**The script pattern:**
1. Set up minimal input (content pack, seed data, test fixture)
2. Run the full pipeline (compile → initialize → execute → output)
3. Print what worked and what didn't at each stage
4. Document every break point with module, line, and error

**The operational rule:**
- No new infrastructure WOs until the canary runs
- Each break point from the canary becomes a work order
- Each integration fix must include a test that exercises the seam (preventing regression)
- The canary script itself becomes a permanent CI artifact

## When to Use

- After completing a batch of module-scoped WOs
- Before transitioning from one project horizon to the next
- When unit test count is growing but confidence in the product isn't
- When architecture audits keep finding "theoretical gaps" — run the system instead

## When Not to Use

- During active development of a single module (unit tests are sufficient)
- When the product path doesn't exist yet (you need components before you can integrate them)

## Real Example

**D&D 3.5e engine, end of H1 WO batch (2026-02-14):**

7 WOs completed across parallel agents: weapon plumbing, RNG protocol extraction, TTS chunking, NarrativeBrief width extension, compile-time cross-validation, runtime narration validation, TTS cold start research. 5,775 unit tests passing.

The operator's directive: "You have 5800 tests that prove individual bricks are solid, but no test that proves the building stands up."

**Predicted break points (confirmed by smoke test):**
- `SPELL_REGISTRY` entries lack `content_id` — Layer B narration permanently dormant
- `CrossValidateStage` not registered in production `WorldCompiler` — cross-validation doesn't run
- `NarrationValidator` not wired into play loop — validation rules exist but never execute
- `content_id` bridge: events → lookup → presentation_semantics: wired but produces `None`

None of these were caught by unit tests. All were caught by trying to cast fireball at a goblin.

## Key Insight

> Constraints produce consistency. Integration produces a product. You need both, and they require different types of work — module-scoped WOs for constraints, cross-cutting WOs for integration.

## Related Patterns

- [Enforcement Hierarchy](ENFORCEMENT_HIERARCHY.md) — Integration seam tests are Tier 1 enforcement
- [Cross-File Consistency Gate](CROSS_FILE_CONSISTENCY.md) — Integration canary is the cross-module version of this
- [Dispatch Self-Containment](DISPATCH_SELF_CONTAINMENT.md) — Integration WOs need broader context than module WOs
