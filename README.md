# Multi-Agent LLM Coordination Framework

**A practical methodology for coordinating multiple AI agents on complex software projects.**

Built and proven on an 8,521+ test, 338-formula deterministic game engine — by a non-technical operator (former chef, English educator) coordinating Claude Opus, Claude Sonnet, and other LLM agents with zero shared memory across 100+ agent sessions over 4+ months.

---

## The Problem

When you use multiple LLM agents to build software, you hit walls that don't exist in solo development:

- **Agents forget everything between sessions.** Every new context window starts from zero.
- **Parallel agents conflict.** Two sessions editing the same file produce merge disasters.
- **Nobody holds the full picture.** No single agent (or human) can keep the entire project in context.
- **Documents drift from reality.** Agent-written docs become stale, contradictory, and self-reinforcing.
- **The human coordinator becomes the bottleneck.** You can't read everything, and you can't tell which agent is right when they disagree.

These aren't theoretical problems. They're what happens on Day 3 of any serious multi-agent project. This framework documents the solutions we discovered by breaking things and codifying the fixes.

---

## What Makes This Different

Most AI coordination guides tell you what to do. This one shows you what went wrong.

**From the proving ground (D&D 3.5e combat engine):**
- 8,521+ automated tests across 9 verification domains
- 338 formulas verified against source rules
- 30 bugs found and categorized into 12 coordination error patterns
- 100+ work orders dispatched across 100+ agent sessions over 4+ months
- 3 of 7 parallel agents silently failed to commit in one dispatch — the fix became a governance pattern

Every pattern in this framework exists because something specific broke. The [failure catalog](case-study/FAILURE_CATALOG.md) has the receipts.

---

## Who This Is For

- **Non-technical builders** using AI agents to construct software ("vibe coders")
- **Developers** scaling from one AI assistant to a fleet of specialized agents
- **Researchers** studying multi-agent LLM coordination
- Anyone who has had an AI agent confidently break something another agent just fixed

---

## Core Principles

### 1. Artifact Primacy
> If it's not in a file, it doesn't exist.

Agents have perfect recall within a context window and zero recall across context boundaries. Conversational knowledge vanishes at session end. Every fact, decision, and status that must survive a context rotation must be pinned to a file.

**Implication:** Your project's files aren't just code and docs — they're the only shared memory your agent fleet has.

### 2. Staged Context Loading
> Reading order matters more than reading volume.

Agents waste context window space when they read files in the wrong order or read everything at once. A defined reading sequence — orientation first, then state, then rules — gets agents operational in minimal context.

**Pattern:** Compass (what is this?) → State (what's done?) → Rules (how do we work?) → Task (what do I do?)

### 3. Dispatch Self-Containment
> Every work order must be executable by an agent with zero prior context.

The work order plus the files it references must contain everything an agent needs. No reliance on "the previous agent will have explained this." Test: could a brand-new agent execute this dispatch using only the dispatch file and linked references?

### 4. Machine Truth Over Prose Truth
> When a script and a document disagree, the script is right.

Agent-written prose drifts from reality within 3-4 handoffs. Machine-generated state (test counts, git status, automated snapshots) doesn't. Build scripts that produce canonical facts, and treat everything else as commentary.

### 5. Protocol Over Memory
> Solve coordination with protocols, not shared state.

You can't give agents shared memory. You can give them protocols — handoff checklists, consistency gates, session scope declarations, structured memo formats. The protocols are the coordination mechanism.

---

## Framework Components

### Patterns
Reusable solutions to specific coordination problems. Each pattern documents the problem, the solution, when to use it, and a real example from the proving ground.

| Pattern | Problem It Solves | Solution |
|---------|------------------|----------|
| [Enforcement Hierarchy](patterns/ENFORCEMENT_HIERARCHY.md) | Corrections don't stick across sessions | 3-tier model: test-enforced > process-enforced > prose-enforced |
| [Staged Context Loading](patterns/STAGED_CONTEXT_LOADING.md) | Agents waste context reading files in wrong order | Defined reading sequence that orients agents in minimal context |
| [Dispatch Self-Containment](patterns/DISPATCH_SELF_CONTAINMENT.md) | Work orders fail because agents lack context | Self-contained work orders that amnesiac agents can execute |
| [Artifact Primacy](patterns/ARTIFACT_PRIMACY.md) | Knowledge lost between sessions | Pin every decision to a file — conversation doesn't survive |
| [Cross-File Consistency Gate](patterns/CROSS_FILE_CONSISTENCY.md) | Partial updates create contradictions | All-or-nothing updates across every file referencing a fact |
| [Session Bootstrap](patterns/SESSION_BOOTSTRAP.md) | Agents start with stale assumptions | Machine-verified truth (tests, git status) before prose documents |
| [Concurrent Session Protocol](patterns/CONCURRENT_SESSION_PROTOCOL.md) | Parallel sessions conflict on shared files | Explicit file ownership with no overlaps between agents |
| [PM Context Compression](patterns/PM_CONTEXT_COMPRESSION.md) | Human coordinator can't read everything | Structured information flow to a bandwidth-limited human |
| [Coordination Failure Taxonomy](patterns/COORDINATION_FAILURE_TAXONOMY.md) | Same mistakes repeat across projects | Categorized catalog of what goes wrong and why |
| [Role Separation](patterns/ROLE_SEPARATION.md) | Agents assume they can do everything | Five-role model with explicit authorities and boundaries |
| [Research-to-Build Pipeline](patterns/RESEARCH_TO_BUILD_PIPELINE.md) | Raw insights produce scope bleed when dispatched directly | Staged conversion: Burst → Research → Brick → Builder WO |
| [Plain English Pass](patterns/PLAIN_ENGLISH_PASS.md) | Non-technical operators can't parse technical debriefs | 3-question translation layer before the technical dump |
| [Debrief Integrity Boundary](patterns/DEBRIEF_INTEGRITY_BOUNDARY.md) | Agent self-reports are trusted without verification | Named trust boundary with verification spectrum and mitigations |
| [Integration Canary](patterns/INTEGRATION_CANARY.md) | Unit tests pass but product doesn't work end-to-end | One script that exercises the full product path before new WOs |
| [Parallel Implementation Parity](patterns/PARALLEL_IMPLEMENTATION_PARITY.md) | Same logic implemented in multiple paths drifts silently | Enumerate all parallel paths on every WO; verify parity before debrief |
| [Sweep Audit Protocol](patterns/SWEEP_AUDIT_PROTOCOL.md) | Sequential WOs can't see system-wide coherence drift | Periodic read-only audits that cross-check subsystems for consistency |
| [Authority Tagging](patterns/AUTHORITY_TAGGING.md) | Community convention ships instead of specification | Every domain-logic WO declares SPEC or POLICY authority — no third type |
| [Post-Debrief Retrospective](patterns/POST_DEBRIEF_RETROSPECTIVE.md) | Builder peripheral observations die in context windows | Mandatory post-debrief question surfaces what the builder noticed |
| [Hidden Assumption Sweep](patterns/HIDDEN_ASSUMPTION_SWEEP.md) | Small questions that reframe architecture go unclassified | 10-question triage protocol converts grenades into named artifacts |
| [Worktree Isolation Protocol](patterns/WORKTREE_ISOLATION_PROTOCOL.md) | Parallel agents collide at filesystem level despite non-overlapping file scope | One git worktree per active builder — isolation at the OS level, not just the work order level |

### Templates
Ready-to-use file templates for implementing the patterns in your own project.

| Template | Purpose | File |
|----------|---------|------|
| [Onboarding Checklist](templates/ONBOARDING_CHECKLIST_TEMPLATE.md) | Reading order + verification steps for new agents | Implements Staged Context Loading |
| [Work Order Dispatch](templates/WORK_ORDER_TEMPLATE.md) | Self-contained task assignment | Implements Dispatch Self-Containment |
| [Audit Work Order](templates/AUDIT_DISPATCH_TEMPLATE.md) | Read-only audit task assignment (never writes code) | Implements Sweep Audit Protocol |
| [Handoff Document](templates/HANDOFF_TEMPLATE.md) | End-of-session knowledge transfer | Implements Artifact Primacy |
| [Session Memo](templates/SESSION_MEMO_TEMPLATE.md) | Structured report to human coordinator | Implements PM Context Compression |
| [Sources of Truth Index](templates/SOURCES_OF_TRUTH_TEMPLATE.md) | Which file is authoritative for which concept | Implements Machine Truth Over Prose |

### Case Study
How these patterns were discovered and proven on a real project.

| Document | Content |
|----------|---------|
| [Project Overview](case-study/PROJECT_OVERVIEW.md) | What was built, by whom, scale and complexity |
| [Failure Catalog](case-study/FAILURE_CATALOG.md) | Real coordination failures, categorized with root causes |
| [Metrics](case-study/METRICS.md) | Quantitative results — formulas verified, bugs found, reclassification rates |
| [Lessons Learned](case-study/LESSONS_LEARNED.md) | What worked, what didn't, what we'd do differently |

---

## Quick Start

**If you're non-technical** and want to learn this methodology from scratch, start with the **[Methodology Guide](METHODOLOGY.md)** — a step-by-step walkthrough written for someone who has a problem, has access to AI agents, and has no idea how to start.

**If you're already running agents** and want to add specific coordination patterns, start here:

1. **Create an onboarding checklist** ([template](templates/ONBOARDING_CHECKLIST_TEMPLATE.md)) — this is the single highest-impact intervention. Define what agents read, in what order.

2. **Create a sources of truth index** ([template](templates/SOURCES_OF_TRUTH_TEMPLATE.md)) — before your second session, decide which file is authoritative for each concept.

3. **Write self-contained work orders** ([template](templates/WORK_ORDER_TEMPLATE.md)) — the moment you start dispatching tasks to agents, the dispatch must stand alone.

4. **Add a session bootstrap script** ([pattern](patterns/SESSION_BOOTSTRAP.md)) — even a simple script that prints git status + test count grounds the agent in reality instead of potentially-stale docs.

5. **Use handoff documents** ([template](templates/HANDOFF_TEMPLATE.md)) — when a session ends, write what the next agent needs to know. This is not optional.

---

## Origin

This framework wasn't designed top-down. It was discovered bottom-up by a non-technical operator coordinating multiple AI agents (Claude Opus, Claude Sonnet, GPT-4) to build a D&D 3.5e deterministic referee engine. Every pattern exists because something broke and the fix got codified into a protocol.

**Project stats (updated 2026-02-27):**
- 8,521+ automated tests (zero regressions across all accepted batches)
- 338 formulas verified against source rules across 9 domains
- 30 bugs found and categorized into 12 error patterns
- 100+ work orders dispatched (fix, feature, research, governance, audit, framework)
- 50+ builder debriefs archived
- 30 research documents produced
- 100+ agent sessions coordinated through file-based protocols
- 25+ delivery batches, each with gate tests as the acceptance arbiter
- Every governance document born from a specific, documented failure

**The operator's background:** Former chef, current English educator, zero programming experience. The methodology emerged because protocols were the only tool available — and it turns out protocols are the right tool for coordinating agents that can't remember yesterday.

---

## Contributing

This is a living framework. If you're using these patterns on your own multi-agent project, your failure cases and solutions make the framework better. Open an issue with:
- What pattern you tried
- What worked or didn't
- What coordination failure you hit that isn't in the taxonomy

---

## License

MIT License — use freely, adapt to your projects, share what you learn.

---

*"Creative in voice, strict in truth, accountable in every outcome."*
