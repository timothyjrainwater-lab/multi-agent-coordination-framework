# Pattern: Role Separation

## Problem

Without defined roles, agents assume they can do everything. A builder session that encounters a gap in the requirements starts researching instead of flagging the gap. A researcher who finds a bug starts fixing it instead of reporting it. An auditor who spots a missing WO starts drafting one instead of noting the gap in their memo.

This produces three failure modes:
1. **Scope bleed** (Coordination Failure Category 3) — agents modify files outside their declared scope
2. **Context waste** — agents spend context window on activities they're not optimized for (a builder doing research burns context that should go to code)
3. **Conflicting outputs** — two agents in different roles produce overlapping or contradictory artifacts because neither knew the other's scope

The root cause is that LLM agents are generalists by default. They'll do whatever seems helpful unless explicitly constrained. Role definitions are those constraints.

## Solution

Define distinct roles with explicit authorities, boundaries, and output formats. Each agent session operates under exactly one role. The role determines what the agent can read, write, and produce.

### The Five Roles

#### 1. Operator (Product Owner)

The only human in the loop. Sets direction, resolves binary decisions, approves dispatches.

| Authority | Boundary |
|-----------|----------|
| Dispatch authority — no WO executes without operator approval | Does not write code |
| Binary decision resolution — DC-01 through DC-N | Does not execute research |
| Priority setting — which WO ships next | Does not draft WOs (PM does) |
| Role assignment — which agent type handles which WO | Does not read full debriefs (reads PM summaries) |

**Key constraint:** The operator is bandwidth-limited. Every artifact that reaches the operator must be compressed to match their available attention. This is why the PM role exists.

#### 2. PM (Project Manager)

The coordination layer. Translates operator intent into agent-executable artifacts. Maintains project state.

| Authority | Boundary |
|-----------|----------|
| Drafts all WOs (research, builder, audit) | Does not execute research (researcher does) |
| Normalizes research into Bricks | Does not write production code (builder does) |
| Maintains briefing, kernel, inbox | Does not audit artifacts (auditor does) |
| Triages and prioritizes incoming artifacts | Does not make binary decisions (operator does) |
| Context compression — translates technical output for operator | Does not dispatch without operator approval |

**Key constraint:** The PM is the bottleneck between research and build. If the PM doesn't normalize a finding into a Brick, it doesn't become a WO. This is by design — it prevents raw research from reaching builders.

#### 3. Builder

Executes builder WOs. Writes code, writes tests, produces debriefs.

| Authority | Boundary |
|-----------|----------|
| Modifies files listed in the WO scope | Does not modify files outside WO scope |
| Runs tests, regenerates fixtures | Does not draft new WOs |
| Writes debrief documenting what was done | Does not perform research |
| Reports mid-session discoveries to PM via debrief | Does not modify governance documents (unless WO explicitly says so) |

**Key constraint:** Builders receive self-contained WOs and produce self-contained debriefs. They never see upstream research, previous session history, or the PM's decision rationale. This is intentional — builders need binary specs, not decision trees.

#### 4. Researcher

Executes research WOs. Reads codebase, reads documentation, reads external sources, produces findings memos.

| Authority | Boundary |
|-----------|----------|
| Reads any file in the codebase | Does not write production code |
| Reads external documentation and specs | Does not draft builder WOs |
| Produces research memos with findings and recommendations | Does not modify existing code |
| May propose schema designs or algorithms in the memo | Does not implement those proposals |

**Key constraint:** Research output is raw material, not finished product. The PM converts research findings into actionable Bricks. Researchers never interact directly with builders.

#### 5. Auditor

Cross-references artifacts against each other. Produces gap analyses, consistency checks, roadmap audits.

| Authority | Boundary |
|-----------|----------|
| Reads all project artifacts (WOs, debriefs, research, code, governance) | Does not modify any file except their output memo |
| Produces audit memos identifying gaps, conflicts, and misalignments | Does not draft WOs (PM does, based on audit findings) |
| May recommend promotions (H2 → H1) or demotions | Does not make prioritization decisions (operator does) |
| Cross-checks debrief claims against diffs and test output | Does not re-execute or modify work |

**Key constraint:** Auditors are read-only except for their output artifact. This prevents the audit from becoming a fix session — the auditor identifies the problem, the PM drafts the fix WO, the builder implements it.

### The Relay Pattern

```
Operator Intent
    → PM drafts Research WO
        → Researcher executes, produces findings memo
            → PM normalizes findings into READY Brick
                → PM drafts Builder WO from Brick
                    → Builder implements, produces debrief
                        → PM compresses debrief for Operator
```

Each handoff is a file. Each file is self-contained. No role depends on conversational context from another role's session.

## Implementation

### Role Declaration in Dispatches

Every WO dispatch includes a role assignment:

```markdown
**Assigned to:** Builder (Opus 4.6)
**Role constraints:** Builder role — modify only files listed in scope, produce debrief on completion
```

Every session memo includes the role:

```markdown
**From:** Auditor (Opus 4.6)
**Role:** Auditor — read-only except output memo
```

### Role Boundary Enforcement

| Enforcement Tier | Mechanism |
|-----------------|-----------|
| Tier 1 (test) | `git diff --stat` post-session — verify only scoped files were touched |
| Tier 2 (process) | WO template includes "Scope Boundaries" and "What NOT to Do" sections |
| Tier 3 (prose) | Role description in onboarding checklist |

### When Roles Need to Cross Boundaries

Sometimes a builder discovers a bug outside their WO scope. Sometimes a researcher finds code that's trivially fixable. The protocol:

1. **Do not cross the boundary.** Note the finding in the debrief or memo.
2. **The PM triages the finding.** If it warrants action, the PM drafts a new WO.
3. **A new session handles it** under the appropriate role.

This feels slow. It prevents scope bleed, parallel collisions, and the coordination failures that cost more time than the extra session.

## When to Use

- Any project with 3+ agent sessions
- Any project where parallel dispatch is planned (role separation is what makes parallel safe)
- Any project where the human coordinator is non-technical (role separation keeps technical complexity inside agent sessions, with only compressed output reaching the human)

## When NOT to Use

- Single-agent projects with one human developer (the human is all roles simultaneously)
- Quick one-off tasks where the overhead of role declaration exceeds the work

## Real Example

The D&D 3.5e project runs all five roles. In a single cycle:
- The **operator** approved 7 dispatch-ready WOs and resolved 3 binary decisions
- The **PM** drafted those 7 WOs from research Bricks, maintained the briefing file, and compressed 12 debriefs into 7-item summaries
- **Builders** executed WOs (RNG protocol extraction, TTS chunking, weapon plumbing, etc.) and produced debriefs
- A **researcher** executed 11 research WOs (RQ-SPRINT-001 through 011) producing findings memos
- An **auditor** cross-referenced the dispatch queue against the roadmap and produced a gap analysis memo

No role touched another role's artifacts. The PM was the only node that read from all roles and wrote dispatches for all roles.

## Anti-Patterns

- **Letting builders research.** A builder who starts reading research docs to "understand context" is burning build-optimized context on research. The WO should contain all needed context.
- **Letting researchers fix bugs.** A researcher who modifies code to "verify their finding" has crossed into builder territory. The fix may conflict with in-flight builder work.
- **Skipping the PM.** Direct operator-to-builder dispatch works for simple tasks but breaks down when the builder needs context the operator can't provide. The PM's value is translating intent into spec.
- **Auditors who draft WOs.** The auditor identifies the gap. The PM decides whether and how to fill it. Combining these roles means the audit is biased toward gaps the auditor knows how to fix, not gaps that matter most.
