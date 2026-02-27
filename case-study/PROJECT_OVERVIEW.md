# Project Overview

**Last Updated:** 2026-02-27

## What Was Built

A deterministic D&D 3.5e referee engine — a software system that adjudicates tabletop RPG combat with mechanical fidelity to the System Reference Document (SRD). The system resolves attacks, damage, conditions, spells, combat maneuvers, movement, terrain, and character progression using the exact formulas published in the Player's Handbook and Dungeon Master's Guide.

## Who Built It

A non-technical operator (former chef, English educator) coordinating multiple AI agents:
- **Claude Opus** — primary builder and verifier
- **Claude Sonnet** — secondary builder for parallel work
- **GPT-4** — research and cross-referencing

The operator wrote zero lines of code directly. All code was produced by AI agents working from dispatched work orders. The operator's role was coordination: writing dispatches, reviewing outputs, resolving ambiguities, and routing agent findings to other agents.

## Scale

| Metric | Value |
|--------|-------|
| Automated tests | 8,521+ (zero regressions across all accepted batches) |
| Verified formulas | 338 across 9 domains |
| Source files | 20+ core resolvers |
| Governance documents | 25+ (each born from a specific failure) |
| Bugs found in verification | 30 (later reduced to ~22 after research cross-referencing) |
| AMBIGUOUS verdicts requiring PM decision | 28 |
| Total work orders dispatched | 100+ (fix, feature, research, governance, audit, framework) |
| Builder debriefs archived | 50+ |
| Research documents produced | 30 |
| Delivery batches completed | 25+ |
| Parallel agent groups (single session) | 7 simultaneous |
| Total agent sessions | 100+ |

## Complexity

D&D 3.5e was chosen specifically because it stress-tests coordination:
- **The rules spec is thousands of pages** across multiple books that sometimes contradict each other
- **Two size modifier tables** (standard attack vs. special/grapple) that apply in different contexts
- **Condition interactions are combinatorial** — prone + helpless + fatigued creates a unique modifier stack
- **The designer published post-hoc rulings** that change RAW interpretations
- **The community has argued about Rules As Written** for 20+ years

This complexity meant the coordination system had to be rigorous. Approximation wasn't acceptable — the engine either implements the formula correctly or it doesn't, and verification can prove which.

## Architecture

The engine uses a frozen packet model (CP-XX) with three layers:
- **Referee Layer** — deterministic rule adjudication (attack resolution, damage, saves)
- **Boundary Layer** — world state management (immutable at non-engine boundaries)
- **Storyteller Layer** — narrative generation (consumes referee output, never modifies state)

RNG streams are isolated (combat/initiative/policy/saves) to enable deterministic replay.

## Timeline Context

The framework was developed over 4+ months of multi-agent coordination — initially intensive (daily sessions) and then sustained across 25+ delivery batches. The operator worked in a constrained network environment with variable API latency, which directly influenced the emphasis on artifact primacy (sessions can drop mid-conversation) and dispatch self-containment (you may not get a chance to clarify).

## Network Environment

This project was built in a constrained network environment with variable latency and intermittent connectivity to AI providers and GitHub. If you're working in a similar environment:

- **API latency is variable.** LLM agent sessions may time out or drop mid-conversation. This makes the handoff protocol and artifact primacy pattern even more critical — if your session drops unexpectedly, pinned artifacts are all that survives.
- **Multiple AI providers may have different accessibility.** Some providers may be reachable, others not, depending on your network configuration. This affects which agents you can use in your fleet and is a real constraint on multi-provider coordination.
- **Context window efficiency matters more.** Higher latency means longer round-trips. Wasting a context window on orientation (because the reading order was wrong) costs more wall-clock time when every API call takes longer.
- **Git push failures.** Remote operations may fail intermittently. If you see SSL errors on git push, try `git config http.sslBackend openssl` in the affected repo.

None of these issues are theoretical. They were encountered during the development of this framework and directly influenced the emphasis on artifact primacy and dispatch self-containment.
