# Project Overview

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
| Automated tests | 5,500+ |
| Verified formulas | 338 across 9 domains |
| Source files | 15+ core resolvers |
| Governance documents | 7+ (each born from a specific failure) |
| Bugs found in verification | 30 (later reduced to ~22 after research cross-referencing) |
| AMBIGUOUS verdicts requiring PM decision | 28 |
| Fix work orders dispatched | 13 covering 30 bugs |
| Parallel agent groups (single session) | 7 simultaneous |

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

The framework was developed over approximately 2 weeks of intensive multi-agent coordination, with the operator working from behind the Great Firewall of China using VPN access to AI providers and GitHub. Network instability directly influenced the emphasis on artifact primacy (sessions can drop mid-conversation) and dispatch self-containment (you may not get a chance to clarify).
