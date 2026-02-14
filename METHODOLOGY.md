# The Methodology: How to Build Software with AI Agents When You Don't Know How to Code

A practitioner's guide for someone who has a problem, has access to AI agents, and has no idea how to start.

---

## Who This Is For

You're a person with a project idea. Maybe you want to build an app, automate a workflow, or create a tool that doesn't exist yet. You've used ChatGPT or Claude or another AI to write code snippets, and it worked — sort of. But now you want to build something real, something with hundreds or thousands of files, something that needs to work reliably and grow over time.

You don't know how to code. Or you know a little, but not enough to build what you're imagining. You've heard you can use AI agents to do the building, but every guide you've found either assumes you're already a developer or waves its hands at the hard parts.

This document is the hard parts.

It's written by a non-technical operator (former chef, English educator) who used this methodology to build a 5,804-test software system by coordinating AI agents through file-based protocols. Every recommendation comes from something that actually happened — usually something that went wrong first.

---

## Before You Start: Three Things to Understand

### 1. AI agents forget everything between sessions

Every time you start a new conversation with an AI agent, it knows nothing about your project. It doesn't remember yesterday's session. It doesn't remember the decisions you made. It doesn't remember the code it wrote. Each session starts from absolute zero.

This is the single most important fact about multi-agent coordination. Everything in this methodology exists because of it.

**What this means for you:** You cannot rely on conversation to coordinate your project. You must rely on files. Every decision, every piece of context, every status update — if it's not written down in a file, it doesn't exist when the next session starts.

### 2. AI agents will do whatever seems helpful unless you tell them not to

If you ask an agent to fix a bug and it notices a different bug nearby, it will probably fix that one too. If you ask it to write a function and it thinks the surrounding code could be better, it will refactor it. This sounds helpful. It isn't.

When you have multiple agents working on the same codebase, an agent that "helpfully" modifies files outside its assigned scope will collide with other agents working on those same files. You'll spend more time untangling the conflicts than the "helpful" fix was worth.

**What this means for you:** You need to tell agents exactly what to do, exactly what files to touch, and explicitly what NOT to do. Constraints are not bureaucracy — they're what makes parallel work possible.

### 3. You are not the coder — you are the coordinator

Your job is not to understand the code. Your job is to understand the project: what needs to happen, in what order, and whether what the agents produced actually works. You're the project manager, not the programmer.

This is a genuine skill. It's not a lesser version of programming. It's a different job — and it's the job that determines whether the project succeeds or fails, because no individual agent can hold the full picture. Only you can.

---

## Phase 1: Your First Session (Day 1)

### Start with one agent and one goal

Don't try to set up infrastructure, governance, or coordination protocols on Day 1. Start with the smallest version of your actual project that an agent can build in a single session.

**Example:** If you're building a game engine, don't start with "build the game engine." Start with "create a function that rolls a 20-sided die and adds a modifier to the result." If you're building an app, don't start with the app — start with one screen that does one thing.

### Tell the agent what you want in plain language

You don't need to use programming terms. Describe what the thing should do, not how it should work internally. The agent knows how to code. You know what the product should do.

```
I want to build a program that tracks my restaurant's daily food costs.
For today, just build the part that lets me enter an ingredient, its quantity,
and its price. Store it in a file. Show me a total at the end.
```

### Ask the agent to write tests

Even if you don't know what tests are, ask for them. Say: "Write tests that prove this works correctly." Tests are how you'll know whether future changes break what you already built. They're the only reliable truth in your project.

After the agent writes the tests, ask it to run them. Look at the output. You should see something like "15 tests passed, 0 failed." That number matters. Write it down.

### Before the session ends: save your context

When you're about to end your first session, ask the agent:

```
Before we stop, write a handoff document that tells the next agent everything
it needs to know to continue this project. Include:
- What we built today
- What files exist and what they do
- What tests exist and whether they pass
- What we decided and why
- What's left to do
```

This handoff document is the first artifact of your project. It's also the most important file you'll create today, because it's the only thing that survives to tomorrow.

---

## Phase 2: Building the Foundation (Days 2-5)

### The "second session problem"

On Day 2, you start a new session. The agent knows nothing. You need to get it oriented quickly. This is where most people hit their first wall — they try to explain the project conversationally, the agent builds a partial mental model, and things go sideways because the agent's understanding doesn't match reality.

**The fix:** At the start of every session, before asking the agent to do anything, have it read your project files in a specific order:

1. **The handoff document** — what was built, what's left
2. **The codebase** — key files, not everything
3. **The tests** — run them, report the count

This is the [Session Bootstrap](patterns/SESSION_BOOTSTRAP.md) pattern. It sounds mechanical. It prevents an entire category of failures where agents build on top of stale or incorrect assumptions.

### Create your Sources of Truth index

By Day 2, you have at least two sessions' worth of files. Some of those files might say conflicting things. Your job: decide which file is the truth for each concept.

Create a simple file that says:

```markdown
# Sources of Truth

- **What the code does:** The test suite (run it, don't read about it)
- **What we've decided:** DECISIONS.md
- **What's been built:** HANDOFF.md (updated each session)
- **What's left to do:** TODO.md
```

This prevents the failure where an agent reads an outdated document and builds on top of wrong information. When two files disagree, this index tells you (and the agent) which one is right.

### Start writing work orders

At some point in the first week, you'll want the agent to do something specific — fix a bug, add a feature, change how something works. Instead of describing it conversationally, write it down in a file.

A work order is just a file that says:

1. **What to do** (specific, concrete)
2. **Why** (what problem this solves)
3. **What files to touch** (and what files NOT to touch)
4. **How to verify it worked** (what test to run)

The [Work Order Template](templates/WORK_ORDER_TEMPLATE.md) has a full format, but even a rough version is better than a conversational description. The work order survives the session. The conversation doesn't.

### Keep a decisions log

Every time you make a decision about the project — "we're using Python, not JavaScript," "the damage formula rounds down, not up," "users log in with email, not username" — write it in a DECISIONS.md file.

Agents will re-ask questions you've already answered. They're not being difficult — they literally don't remember. The decisions log is how you answer without repeating yourself: "Read DECISIONS.md, item 7."

---

## Phase 3: Scaling Up (Week 2+)

### When one agent isn't enough

At some point your project gets big enough that a single agent session can't hold the full context. You'll notice this when agents start making mistakes because they didn't read a relevant file, or when sessions end before the work is done because the agent used up its context window reading files.

This is when you need multiple agents working in parallel. And this is where coordination becomes the actual work.

### The rules for parallel agents

If you're going to have multiple agents working on your project simultaneously, you need three things:

**1. File ownership.** Each agent gets an explicit list of files it's allowed to modify. No overlaps. If Agent A is working on the login system and Agent B is working on the database, neither touches the other's files. This prevents merge conflicts and ensures each agent's changes are independent.

**2. Self-contained work orders.** Each agent gets a work order that contains everything it needs. No "check with the other agent" or "this depends on what Agent B decides." Each work order stands alone. If Agent B's work order requires a decision that Agent A hasn't made yet, Agent B shouldn't have been dispatched yet.

**3. Post-completion verification.** When an agent says "done," check the actual output. Run `git diff` to see what files changed. Run the tests. 43% of our parallel agents reported "task complete" with zero code changes on disk. The agent believed it was done. The files said otherwise.

### Introduce roles

Once you're running parallel sessions, you need to separate what different agents do. Not every agent should do everything. Define roles:

- **Builder agents** write code and tests. They get work orders, they execute, they report what they did.
- **Researcher agents** investigate questions. They read code, read documentation, and produce findings — but they don't modify code.
- **Auditor agents** check other agents' work. They cross-reference outputs against requirements and flag discrepancies.

You — the operator — are the coordinator. You don't write code. You don't research. You set direction, make decisions, and route information between agents.

Why separate roles? Because an agent that's building code shouldn't also be researching alternatives. That wastes its context window on exploration when it should be spending it on implementation. And an agent that's auditing shouldn't also be fixing what it finds — the fix might conflict with another agent's in-progress work.

See the [Role Separation](patterns/ROLE_SEPARATION.md) pattern for the full model.

### The PM role (the role you'll grow into)

As your project scales, you'll find that you spend most of your time doing the same things:

- Reading what agents produced
- Deciding what needs to happen next
- Writing work orders for the next round of agents
- Compressing technical output into something you can reason about

This is the PM role. In the early days, you're both the operator (decision maker) and the PM (coordinator). As the project grows, you may want to dedicate an agent session specifically to PM work — reading debriefs, organizing findings, drafting work orders for your approval.

The key insight: **you build understanding by reading agent outputs, not code.** You never need to understand the codebase at the code level. You need to understand what was done, whether it worked, and what it means for the project's direction. Agent debriefs, structured correctly, give you this.

---

## Phase 4: The Research-to-Build Pipeline (When Your Ideas Get Complex)

### The problem with jumping straight to building

Eventually you'll have an idea that's too complex to dispatch directly to a builder. "Add voice input to the system." "Make the app work offline." "Support multiplayer."

If you dispatch these as builder work orders, the builder will have to make dozens of design decisions during the build. Some of those decisions will be wrong. You'll discover they're wrong after the code is written. Rework follows.

### The pipeline

Instead of going straight from idea to build, run your ideas through a pipeline:

**Step 1 — Capture the idea.** Write it down in an intake file. One sentence. Don't try to spec it out yet. ("Voice input should feel reliable, not experimental.")

**Step 2 — Research it.** Draft a research work order that asks a specific question: "What is the cold start time of the TTS engine? What are the options for reducing it?" Send a researcher agent to investigate. The output is a findings memo — not code, not a plan, just findings.

**Step 3 — Normalize the findings into a "Brick."** This is the critical conversion step. You (or a PM agent) take the research findings and produce a structured packet:
- **Target Lock:** One sentence describing the end state
- **Binary Decisions:** Choices you need to make (with exactly two options each)
- **Contract Spec:** The technical specification
- **Implementation Plan:** The ordered list of builder work orders

**Step 4 — Resolve the decisions.** Read the binary decisions. Pick one option for each. You don't need to understand the technical details — the Brick should present each option in terms of tradeoffs you can evaluate. ("Option A: faster but uses more memory. Option B: slower but runs on any machine.")

**Step 5 — Dispatch builders.** Once decisions are resolved, draft builder work orders from the Brick. Each work order is self-contained. The builder never sees the upstream research — only the spec.

This pipeline sounds heavy. For simple features, skip it and go straight to a builder work order. For anything with design ambiguity — anything where you'd say "I'm not sure how this should work" — the pipeline prevents expensive rework.

See the [Research-to-Build Pipeline](patterns/RESEARCH_TO_BUILD_PIPELINE.md) pattern for the full implementation.

---

## Phase 5: Governance (What You Build After Things Break)

### Why governance documents exist

Every governance document in this framework was created after a failure. The cross-file consistency gate was created after an agent updated one file but forgot the other three. The session bootstrap pattern was created after an agent confidently built on top of stale information. The post-completion verification gate was created after three agents said "done" with zero code changes.

You will have failures. The question is whether you codify the fix into a protocol that prevents recurrence, or whether you have the same failure again in two weeks.

### The governance documents you'll create (and when)

You don't need to create these upfront. Create each one when you need it:

**After your second session:**
- Sources of Truth index (which file is authoritative for which concept)
- Handoff template (standardize what agents write at session end)

**After your first parallel dispatch:**
- Work order template (standardize task assignments)
- Post-completion verification checklist (check `git diff` before accepting results)

**After your first coordination failure:**
- Whatever protocol prevents that specific failure from happening again. Write it down, put it in a governance file, and reference it in future work orders.

**After you have 5+ governance files:**
- An onboarding checklist (reading order for new agents) — so agents don't waste context reading governance documents in random order

### The enforcement hierarchy

Not all rules are created equal. You'll discover this when agents violate a rule you thought was clear:

- **Tier 1: Test-enforced.** If breaking the rule causes a test to fail, the agent can't ship the violation. Compliance rate: ~100%.
- **Tier 2: Process-enforced.** If breaking the rule is caught by a template section or a checklist step, the agent usually follows it. Compliance rate: ~70-80%.
- **Tier 3: Prose-enforced.** If the rule is written in a document but not enforced by anything, agents violate it routinely. Compliance rate: ~40-60%.

When a rule matters, push it up the hierarchy. A file-count cap written in a README will be violated. A file-count cap enforced by a pre-commit test won't be.

See the [Enforcement Hierarchy](patterns/ENFORCEMENT_HIERARCHY.md) pattern.

---

## Phase 6: Staying in Control at Scale

### The briefing file

When your project reaches 20+ sessions and dozens of artifacts, you'll lose the ability to hold the full picture in your head. This is normal. You need a single file that gives you the current state of the project in under 5 minutes of reading.

Create a rolling briefing file. Update it after each cycle (batch of dispatches + results). It should answer:

1. What's in progress right now?
2. What's blocked and why?
3. What finished since last briefing?
4. What decisions do I need to make?
5. What's coming next?

This file is your cockpit. Everything else is reference material.

### Reading debriefs (your primary learning channel)

You build understanding of your project by reading what agents did, not by reading code. Agent debriefs — structured reports of what they built, what worked, what didn't, and what they're concerned about — are your primary learning channel.

Require a structure that includes a plain-language section:

1. **What problem did this solve?** (1-2 sentences, no technical jargon)
2. **What does it actually do?** (1-2 sentences, describe the mechanism in everyday language)
3. **Why should anyone care?** (1-2 sentences, describe the user-facing impact)

Below the plain-language section, the agent can dump all the technical detail it wants. But the plain-language section is for you, and it's what lets you make informed decisions about the project's direction without needing to understand the code.

See the [Plain English Pass](patterns/PLAIN_ENGLISH_PASS.md) pattern.

### The debrief trust problem

Agents self-report. There's no guarantee their self-report is accurate. An agent can say "all tests pass" when they don't. An agent can say "I considered three approaches" when it tried one. An agent can say "no concerns" when it didn't think about potential problems.

You need to distinguish between what you can verify and what you have to trust:

- **Machine-verifiable:** Test pass counts, `git diff` output, commit hashes. These are facts.
- **Process-verifiable:** Did the agent follow the template? Are all sections filled out? Did it reference the correct files? These are checkable.
- **Prose-only:** The agent's retrospective, methodology notes, concerns section. These are trust.

Expand the machine-verifiable surface whenever possible. Require agents to include `git diff --stat` output in their debriefs so you can see exactly what files changed. Require test output so you can see pass/fail counts. The more structured data you require, the harder it is for incomplete work to hide behind prose.

See the [Debrief Integrity Boundary](patterns/DEBRIEF_INTEGRITY_BOUNDARY.md) pattern.

---

## The Mistakes You'll Make (and How to Recover)

These are not hypothetical. They all happened.

### Mistake 1: Trusting "task complete"

An agent says it's done. You move on. Later you discover nothing was actually written to disk. Three out of seven agents did this to us in a single batch.

**Recovery:** Always check `git diff` after an agent reports completion. If the diff is empty and it shouldn't be, the agent didn't actually do the work.

### Mistake 2: Letting documents drift

A document says the project has 98% confidence. A verification pass reveals a 5.5% error rate. The document was written once and never updated.

**Recovery:** Date every document. Run machine verification (tests, diffs) before trusting any prose claim. When a document makes a quantitative claim, verify it against the actual data.

### Mistake 3: Not enforcing scope boundaries

An agent fixes a bug and also refactors three nearby functions. Another agent, working in parallel, was about to modify those same functions. Conflict.

**Recovery:** Work orders should include a "What NOT to Do" section. If an agent finds something outside its scope, it notes it in the debrief. The PM drafts a new work order. The agent does not fix it.

### Mistake 4: Assuming research is the same as building

You learn about a technology, get excited, and dispatch a builder to implement it. The builder makes design decisions you didn't anticipate. The result doesn't match what you wanted.

**Recovery:** Separate research from building. Research produces findings. The PM converts findings into a spec. The builder implements the spec. If the spec requires decisions, the operator makes them before the builder starts.

### Mistake 5: Growing the inbox without a triage system

You produce work orders, research memos, debriefs, audit findings. The inbox hits 23 items. You can't process them fast enough. Important items get buried.

**Recovery:** Create a briefing file immediately. Triage everything into a prioritized queue. Process the queue in order. Archive completed items so the inbox stays manageable.

---

## The Toolkit (Minimum Viable Setup)

If you're starting today, here's the minimal set of files to create:

| File | Purpose | When to Create |
|------|---------|----------------|
| `HANDOFF.md` | What the next agent needs to know | End of Day 1 |
| `DECISIONS.md` | Every decision made, with rationale | Day 1 |
| `SOURCES_OF_TRUTH.md` | Which file is authoritative for what | Day 2 |
| `ONBOARDING_CHECKLIST.md` | Reading order for new agent sessions | When you have 5+ project files |
| Work orders (per task) | Self-contained task assignments | When you start dispatching specific tasks |
| `BRIEFING.md` | Rolling project status | When you can't hold the full picture in your head |

Don't create governance documents you don't need yet. Each one should be born from a specific problem you encountered. If you haven't had the failure, you don't need the protocol.

---

## How to Know It's Working

You're doing this right when:

- **A brand-new agent can start working within its first few messages** because the onboarding checklist and handoff document tell it everything it needs.
- **You can dispatch work to multiple agents in parallel** and their outputs don't conflict.
- **You can explain what your project does and where it's going** without understanding the code — because the debriefs and briefing file give you that understanding.
- **When something breaks, you know why** and you can point to the governance document that should have prevented it (or create one that will prevent it next time).
- **Your test count goes up over time** and never goes down. Tests are the only reliable measure of progress. Everything else is commentary.

You're doing this wrong when:

- You spend most sessions re-explaining the project to agents.
- Agents keep modifying the same files and creating conflicts.
- You have documents that contradict each other and you're not sure which is right.
- An agent says "done" and you have no way to verify that's true.
- You're afraid to make changes because you don't know what will break.

---

## One Last Thing

This methodology was built by someone who couldn't code, for people who can't code. It works. It produced a system with nearly 6,000 tests, verified against primary sources, with every significant coordination failure cataloged and resolved.

But it's not magic. You'll still have sessions that go sideways. Agents will still misunderstand your instructions. Work orders will still have gaps. The difference is that every failure becomes a protocol, and every protocol makes the next session better.

The compound effect is real. By Day 20, your coordination infrastructure is so robust that new agents become productive in minutes, not hours. By Day 30, you're dispatching 7 parallel agents and recovering from failures in the same session they occur. The early investment in files, protocols, and governance pays for itself many times over.

Start small. Write things down. Trust the files, not the conversation. Build the governance when it hurts, not before.

That's the methodology.
