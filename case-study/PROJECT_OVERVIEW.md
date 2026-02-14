# Project Overview

**Last Updated:** 2026-02-14

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
| Automated tests | 5,804+ (15 pre-existing failures, 0 regressions) |
| Verified formulas | 338 across 9 domains |
| Source files | 15+ core resolvers |
| Governance documents | 10+ (each born from a specific failure) |
| Bugs found in verification | 30 (later reduced to ~22 after research cross-referencing) |
| AMBIGUOUS verdicts requiring PM decision | 28 |
| Total work orders dispatched | 37 (fix, feature, research, governance, audit, framework) |
| Builder debriefs archived | 14 |
| Research documents produced | 30 |
| Parallel agent groups (single session) | 7 simultaneous |
| Total agent sessions | 45+ |

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

## Network Environment

This project was built behind the Great Firewall of China, accessing GitHub, AI APIs, and documentation through a VPN. If you're working in a similar restricted network environment:

- **SSL certificate revocation checks may fail.** Git pushes can fail with `CRYPT_E_REVOCATION_OFFLINE`. Fix: `git config http.sslBackend openssl` in the affected repo.
- **API latency is variable.** LLM agent sessions may time out or drop mid-conversation. This makes the handoff protocol and artifact primacy pattern even more critical — if your session drops unexpectedly, pinned artifacts are all that survives.
- **Multiple AI providers may have different accessibility.** Some providers are reachable, others aren't, depending on your VPN routing. This affects which agents you can use in your fleet and is a real constraint on multi-provider coordination.
- **Context window efficiency matters more.** Higher latency means longer round-trips. Wasting a context window on orientation (because the reading order was wrong) costs more wall-clock time when every API call takes longer.

None of these issues are theoretical. They were encountered during the development of this framework and directly influenced the emphasis on artifact primacy (sessions can drop) and dispatch self-containment (you may not get a chance to clarify).
