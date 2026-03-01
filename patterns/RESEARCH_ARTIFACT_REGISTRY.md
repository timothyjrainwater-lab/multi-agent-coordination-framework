# Pattern: Research Artifact Registry

## Problem

A project accumulates research documents — sprint memos, OSS evaluations, design theses, strategy papers. Each one costs real context window tokens and operator time to produce. Each one contains decisions, findings, and recommendations that should feed the build queue.

But without explicit tracking, research becomes a dead artifact: produced, filed, and never converted into action. The next agent session references "the research we did on X" without anyone verifying that the research actually drove a decision. The decisions it was supposed to inform get made again from scratch — or silently get made wrong because the research findings weren't applied.

**The damage is invisible.** The documents exist. The project looks like it has research coverage. But the research never closed the loop — it never became a backlog entry, a WO, or a confirmed REFERENCE-ONLY decision. Tokens were spent; value was not extracted.

This is distinct from the [Research-to-Build Pipeline](RESEARCH_TO_BUILD_PIPELINE.md), which governs the *forward* flow: how new research is commissioned and converted. This pattern governs the *backward* audit: how existing research is verified to have actually been consumed.

## Solution

### The Registry

Maintain a single master file that lists every research artifact in the project:

```markdown
| Artifact | Type | Date | Sweep Status | Backlog Conversion | Notes |
|---|---|---|---|---|---|
| `docs/voice_research.md` | OSS Sprint | 2026-01-15 | SWEPT+LEDGERED | WO-VOICE-01 dispatched | Tier 1 libs confirmed viable |
| `docs/db_comparison.md` | Strategy | 2026-01-20 | PARTIAL | COMPLETE for sections 1-3; section 4 too large | Full audit deferred |
| `docs/competitor_analysis.md` | Research | 2026-01-22 | NOT-SWEPT | — | Not yet reviewed |
```

**Disposition options:**
- `SWEPT+LEDGERED` — Every section has been read, findings extracted, and all items either converted to backlog entries or explicitly marked REFERENCE-ONLY
- `PARTIAL` — Read begun but incomplete (usually large files). Reason for deferral must be documented
- `NOT-SWEPT` — Not yet reviewed. This is a queue item, not an acceptable resting state
- `SUPERSEDED` — Replaced by a newer artifact; original no longer action-relevant

### The Sweep Lifecycle

Every research artifact enters at `NOT-SWEPT` and must exit at `SWEPT+LEDGERED`, `PARTIAL` (with documented reason and owner), or `SUPERSEDED`. No artifact stays `NOT-SWEPT` indefinitely.

**A research artifact with no backlog entries is a dead artifact.** One of two things should be true: the research produced action items (backlog entries, WO candidates, design decisions, confirmed REFERENCE-ONLY resolutions), or it was explicitly marked SUPERSEDED with a reason. Neither being true means the research was noise — and noise is expensive.

### Section Ledger Format

When sweeping a research artifact, the auditor produces a section ledger entry:

```markdown
SOURCE: docs/voice_research.md
SECTION LEDGER:
  §1 (TTS options) → SWEPT: 3 libraries evaluated. Decided: Kokoro (local). WO-VOICE-TTS-01 queued.
  §2 (latency benchmarks) → SWEPT: Benchmarks confirmed Kokoro meets spec. No new WO needed.
  §3 (API fallback options) → REFERENCE-ONLY: No API fallback planned. Decision logged.
  §4 (enterprise licensing) → DEFERRED: Out of scope for current phase. Low priority.
FILE RESULT: 1 WO queued (WO-VOICE-TTS-01). 1 decision confirmed (no API fallback). 1 section deferred.
```

This format makes the sweep auditable. Anyone can see what was read, what decision each section drove, and whether the sweep was complete.

### The Source Artifact Verification Gate

Before dispatching any WO that claims to derive from research, the PM must open the source artifact and read the relevant section. This prevents "ghost research" — WOs that cite findings that don't actually exist in the referenced document, or that misremember what the research concluded.

**The gate:** PM reads source → quotes or paraphrases the specific finding → then drafts the WO. If the PM cannot locate the finding in the source artifact, the WO is blocked until the source is confirmed.

This is a one-step check. It costs one Read tool call per WO and prevents an entire class of build errors where agents implement something the research specifically recommended against.

### Registry Sweep Cadence

Sweep the registry at the same cadence as subsystem code audits, or at session boundaries when research-derived WOs are being drafted:

1. Identify all `NOT-SWEPT` and `PARTIAL` artifacts
2. For each, either: execute a section ledger sweep or explicitly extend the deferral with a documented reason and expected sweep date
3. Update the registry file
4. Any research with `NOT-SWEPT` status older than 2 project phases is a governance finding — it means research was commissioned but never consumed

## Implementation

### Registry File Structure

```markdown
# Research Artifact Registry

**Purpose:** Master tracking list of all research documents → sweep status → backlog disposition.
A sweep is not complete until the FILE RESULT column is populated with a ledger entry.

## [Category: e.g., OSS Research]

| Artifact | Type | Date | Sweep Status | Backlog Conversion | Notes |
|---|---|---|---|---|---|
| ... | ... | ... | NOT-SWEPT | — | Pending sweep |
```

**One registry file per project.** Don't split by category — the value of the registry is seeing all research in one place and spotting `NOT-SWEPT` items easily.

### Who Owns the Sweep

The **PM** owns the registry — maintaining it, initiating sweeps, and tracking status. The **auditor** (if the project has one) executes sweeps and produces section ledgers. If the project has no auditor role, the PM executes sweeps.

**The builder does not sweep research.** The builder receives WOs derived from research, not raw research documents. Builders reading unfiltered research produces the same problems as the [Research-to-Build Pipeline](RESEARCH_TO_BUILD_PIPELINE.md) is designed to prevent.

### Minimum Registry Entry

Every entry needs: artifact path, type (OSS Sprint / Strategy / Design / Reference), date created, sweep status. Everything else (backlog conversion, notes) is filled in during the sweep.

An empty `Backlog Conversion` column on a `NOT-SWEPT` item is expected. An empty column on a `SWEPT+LEDGERED` item is a registry error — the sweep must produce a FILE RESULT.

## When to Use

- Any project that produces research documents (OSS evaluations, design memos, strategy papers, competitor analyses)
- Before dispatching a batch of WOs — verify the research those WOs derive from is in the registry and has been swept
- Whenever an agent or human says "based on our earlier research" — verify that research has a registry entry and a FILE RESULT

## When NOT to Use

- One-off exploratory sessions where the "research" is just a conversation — if it didn't produce a document, there's nothing to track
- Internal draft artifacts that were superseded before anyone acted on them — mark SUPERSEDED and move on
- Research artifacts that are themselves specifications (design contracts, schema specs) — those belong in the spec registry, not the research registry

## Anti-Patterns

- **Research without a registry entry.** If it's research, it needs an entry. The registry exists to prevent research from evaporating — but only if research that was actually done shows up in it.
- **Marking `SWEPT+LEDGERED` without a FILE RESULT.** A sweep with no documented outcome is not a sweep. If the artifact contained nothing actionable, FILE RESULT should say so explicitly: "FILE RESULT: Nothing new to add. All references tracked."
- **`PARTIAL` as a permanent state.** PARTIAL means "deferred for documented reason." It is not a final state. Every PARTIAL entry should have a deferred-by date and an expected sweep date. PARTIAL items older than two project phases are a governance finding.
- **Sweeping research without reading it.** The source artifact verification gate exists for this reason. An agent that marks a research artifact SWEPT+LEDGERED without reading the full document is producing false coverage.
- **Building from unswept research.** If a WO cites research that is `NOT-SWEPT` in the registry, the WO is derived from unverified findings. The research may say something different than the PM remembers. Sweep first, dispatch second.

## Real Example

A project had accumulated 19 research artifacts across 5 categories — OSS evaluations, design theses, strategy memos, specs, and PHB mechanic inventories. Most were listed in a registry but marked `NOT-SWEPT` or `PARTIAL`.

An auditor swept all 19 in a single session using the section ledger format. Results:

- 13 artifacts → `SWEPT+LEDGERED`: 2 findings closed, 3 new LOW findings added, multiple backlog entries confirmed
- 6 artifacts → `PARTIAL`: files too large to fully read in one context window; specific sections read, remainder deferred with documented reason
- 1 critical finding: one named OSS library had been cited as "requiring a WO to evaluate" across multiple sessions. The sweep confirmed the research had already resolved this — the library was `REFERENCE-ONLY` with no WO needed. The finding was closed. No rework was done; the research just hadn't been consumed.

The sweep also surfaced a referenced artifact (`docs/research/OSS_SHORTLIST.md`) that existed in a footnote but was not in the registry. Added as `UNTRACKED` — a new finding that needed resolution.

**Total value: 1 WO cancelled (would have re-researched something already resolved), 3 new findings correctly categorized, 6 deferred items documented with explicit reasons rather than silently ignored.**
