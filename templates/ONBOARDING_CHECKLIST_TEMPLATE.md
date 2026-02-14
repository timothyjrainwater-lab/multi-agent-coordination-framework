# Onboarding Checklist Template

Use this template to create an onboarding checklist for your project. The checklist defines what an agent reads, in what order, to become operational with minimal context waste.

---

## Instructions

Copy the template below into your project as `AGENT_ONBOARDING_CHECKLIST.md`. Replace bracketed placeholders with your project's actual files and commands. Remove sections that don't apply.

---

## Template

```markdown
# Agent Onboarding Checklist

**Purpose:** Get any agent operational in minimum context. Read in order. Do not skip steps.

## Phase 1: Orient (What is this project?)

- [ ] Read `[PROJECT_COMPASS.md or README.md]` — project purpose, architecture, key terms
- [ ] Read `[SOURCES_OF_TRUTH.md]` — which file is authoritative for which concept

**After Phase 1, the agent should know:** What the project does, what its major components are, and where to find canonical information.

## Phase 2: Ground (What is the current state?)

- [ ] Run session bootstrap:
  ```bash
  git status
  git log --oneline -5
  [test command] 2>&1 | tail -3
  ```
- [ ] Read `[PROJECT_STATE_DIGEST.md]` — current milestone, blockers, recent decisions
- [ ] Cross-check: Does the bootstrap output match the state digest? If not, the digest is stale — trust the bootstrap.

**After Phase 2, the agent should know:** What's done, what's in progress, what's blocked, and whether the project is healthy.

## Phase 3: Rules (How do we work here?)

- [ ] Read `[AGENT_DEVELOPMENT_GUIDELINES.md]` — coding conventions, naming rules, common pitfalls
- [ ] Read `[COHERENCE_DOCTRINE.md or ARCHITECTURE.md]` — architectural constraints that cannot be violated
- [ ] Read `[AGENT_COMMUNICATION_PROTOCOL.md]` — how to escalate, what to do when stuck

**After Phase 3, the agent should know:** What it's allowed to do, what it must not do, and how to ask for help.

## Phase 4: Task (What do I do now?)

- [ ] Read the specific dispatch or work order assigned to this session
- [ ] Verify: Is the dispatch self-contained? Can I execute it using only the dispatch + files it references?
- [ ] If the dispatch references files, read those files now
- [ ] Begin work

**After Phase 4, the agent is operational.**

## Verification Gates

Before starting work, verify these assertions:

- [ ] I can name the project's primary test command
- [ ] I know which branch I'm on and whether it's clean
- [ ] I know the 3 files I must never edit without explicit instruction: `[list critical files]`
- [ ] I know where to write my session output (handoff file, memo, or dispatch response)

## Emergency: If Context Runs Low

If the context window is filling up before the task is complete:

1. Write a handoff document ([template](HANDOFF_TEMPLATE.md)) with current state
2. Commit any completed work
3. Report to the coordinator what's done and what's remaining
4. Do NOT rush to finish — a clean handoff is better than a broken completion
```

---

## Customization Notes

- **Phase 1 should be 2-3 files maximum.** If the agent needs to read more than 3 files to understand the project, your orientation documents need consolidation.
- **Phase 2 must include machine verification.** A session bootstrap command is non-negotiable. Without it, the agent may operate on stale assumptions.
- **Phase 3 scales with project complexity.** A simple project might have one rules file. A complex project might have 3-4. Don't exceed 4 — if you have more, consolidate.
- **Phase 4 is always exactly one dispatch.** The agent does one thing per session. If you need multiple things done, dispatch multiple sessions.