# Lessons Learned

What worked, what didn't, and what we'd do differently. Derived from 4+ months of multi-agent coordination on a 8,500+ test codebase across 25+ delivery batches and 100+ work orders.

---

## What Worked

### 1. Self-contained dispatches are the single highest-impact practice

Every fix WO included: file path, line number, SRD citation, exact fix description, test requirements, and commit message format. An agent with zero prior context could pick up any WO and execute it. This is what made 7-way parallelism possible.

**The test:** Could a brand-new agent execute this dispatch using only the dispatch file and the files it references? If yes, the dispatch is self-contained. If no, it will fail.

### 2. Verification before fixing prevents wasted work

The 338-formula verification pass found 30 bugs. The research cross-reference pass reclassified ~8-10 of those as design decisions. Without the cross-reference, agents would have "fixed" intentional behavior, creating new bugs.

**The lesson:** Verify what's actually wrong before dispatching fixes. A bug report that hasn't been cross-referenced against design decisions is unreliable.

### 3. Machine truth grounds agents in reality

Running `pytest` at session start and `git diff` before committing caught problems that no amount of document reading would reveal. The session bootstrap pattern saved at least 3 wasted work sessions by catching stale assumptions early.

### 4. The human coordinator doesn't need to understand the code

The operator (a non-programmer) coordinated 45+ agent sessions by understanding the coordination problem, not the codebase. Writing dispatches, reviewing diffs, resolving ambiguities, and routing findings across agents — none of this requires knowing Python. It requires knowing project management.

### 5. Artifact primacy works as advertised

Every fact that survived a context boundary was in a file. Every fact that was lost wasn't. There are zero exceptions to this rule in the project's history. Conversational knowledge that isn't pinned to an artifact is assumed lost — and that assumption has been correct every time.

### 6. Debriefs as a learning loop

The operator builds system understanding by reading debriefs, not code. The 4-pass debrief format (Plain English → Full Dump → PM Summary → Retrospective) is the mechanism. The Plain English pass translates what the agent did into terms the operator can reason about. Over 50+ debriefs, the operator developed a mental model of the system's architecture, its weak points, and its trajectory — without ever reading a line of Python.

**The lesson:** Debriefs aren't status reports for tracking task completion. They're the operator's primary learning channel. Optimizing debrief quality directly improves the operator's decision-making on future dispatches.

---

## Methodology Lessons — Second Phase (ML-001 through ML-006)

These lessons emerged from 25+ delivery batches (Batches M through V and beyond) on a codebase that had grown to 8,500+ tests. They represent a second tier of learning — operational failures that only become visible at scale.

### ML-001: "Filed" Is Not "Accepted" — Gate Tests Are the Arbiter

**What happened:** Multiple work orders were reported as complete in builder debriefs. A subsequent audit found the code absent from the codebase — fields missing, handlers not wired, functions not in the expected locations. The debrief was accurate as a description of intended work. The code wasn't there.

**Root cause:** Builders write accurate descriptions of work they believe they completed. Without a gate run at verdict time, a debrief is self-reported. "Filed" means the builder believes it's done. "Accepted" means gate tests confirmed it.

**Rule:** No WO status upgrades to ACCEPTED without a gate run. If a WO has no gate tests, it cannot be accepted — it must be held until a subsequent gate run validates it.

### ML-002: Post-Debrief Loose Threads Must Be Actively Solicited

**What happened:** Coupling risks and structural observations surfaced after a formal debrief — only because a builder was still looking at the seams. The formal debrief format is oriented toward delivery. Drift risks and coupling issues require a different lens, one that activates after the delivery pressure is off.

**Root cause:** Builders orient the debrief toward what was done. Structural risks require the question "what did you notice?" rather than "what did you do?"

**Rule:** After every accepted debrief, ask: "Anything else you noticed outside the debrief?" File any findings before closing. (See POST_DEBRIEF_RETROSPECTIVE pattern.)

### ML-003: Coverage Audit Maps Are Not Ground Truth — Verify the Gap Before Writing Code

**What happened:** Work orders were generated from coverage audits that flagged features as missing. Builders wrote implementation code. Gate tests confirmed the features were already fully implemented — the audits had misread the existing code.

**Root cause:** Coverage audits are generated from code inspection at a point in time. They can misread existing implementations, especially when function names don't obviously signal their scope. A WO generated from an audit inherits the audit's confidence level, not the codebase's actual state.

**Rule:** Any WO targeting "missing" functionality must include an Assumptions to Validate step that explicitly confirms the gap exists before writing code. If the builder finds the feature is already implemented: gate tests validate existing behavior, zero production changes. Not a failure — correct methodology.

### ML-004: Regression Spirals Burn Context Without Producing Output

**What happened:** A builder completed all implementation and its own targeted gate tests. The builder then ran the full regression suite, hit failures unrelated to its WO, and entered a retry loop — 28+ tool calls, never filed a debrief, burned context without producing output. The operator closed the session manually after confirming the code was complete.

**Root cause:** The prompt implied the builder must achieve zero failures before filing. The full suite had pre-existing failures. An agent that doesn't know this will retry indefinitely. No retry cap was specified; no stop rule was given.

**Rule:** Every batch dispatch includes (1) a retry cap ("fix once, re-run once — if still failing, record and stop"), (2) the known pre-existing failure count ("N pre-existing failures — do not treat these as regressions"), and (3) a separate regression agent for the full suite after all WOs in a batch land.

### ML-005: PM Commits Can Sweep Staged Engine Code

**What happened:** A builder had staged production code changes but not yet committed. The PM ran `git add <pm files> && git commit` to land PM-side work. Git committed ALL staged content — including the builder's staged engine files. The engine code appeared in a PM commit, corrupting the audit trail.

**Root cause:** `git commit` (without `-a`) commits all staged content, not only files explicitly added in the most recent `git add`. A parallel builder staging without committing creates a shared staging area trap.

**Rule:** PM must run `git status` before every commit. If any production files appear in the staging area, do NOT proceed. Signal the builder to commit their staged changes first, then PM commits separately.

### ML-006: Missing Authority Tagging Lets Spec Errors Ship as Engine Behavior

**What happened:** A WO was dispatched to implement a calculation rule. The rule had a widely-used community variant that differed from the written specification. The WO dispatch had no authority tag. The builder used the community variant — plausible, tests passed. The deviation was discovered at debrief by a reviewer who read the original specification directly. Correcting it required a builder WO and downstream effects.

**Root cause:** The WO spec had no obligation to declare what authority the implementation should follow. The builder had no signal that this was a contested point. The ambiguity flowed through undetected because it was never made visible.

**Rule:** Every WO touching domain logic with any community dispute (multipliers, stacking rules, edge cases) must include an authority tag in the spec: SPEC (cite source + location) or POLICY (cite rationale + operator sign-off). No third type. (See AUTHORITY_TAGGING pattern.)

---

## What Didn't Work

### 1. Trusting agent completion reports

43% of parallel agents (3/7) reported "task complete" with zero code changes on disk. The agents' internal assessment of completion didn't match the external reality. Without the commit review step (checking `git diff` for actual changes), three WOs would have been marked done with nothing to show for it.

**What we'd do differently:** Require agents to include the `git diff --stat` output in their completion report. If the diff is empty, the agent must explain why.

### 2. Prose-enforced rules don't stick

The PM inbox had a 10-item cap documented in its README. The inbox reached 23 items. The rule existed at Tier 3 (prose) and was violated 2.3x over. Rules that aren't enforced by tests or process conventions are suggestions, not rules.

**What we'd do differently:** Start with Tier 1 enforcement for any quantitative rule (file count caps, naming conventions). Promote rules up the tier hierarchy when they're violated, not when someone notices they should have been enforced all along.

### 3. Schema change impact estimation

WO-FIX-03 was scoped to touch 3 files but actually required 6. The WO author traced direct consumers (attack resolvers) but missed the infrastructure layer (serialization, aggregation, tests). This pattern repeated — schema changes always touch more files than expected.

**What we'd do differently:** Schema-change WOs include a mandatory cascade checklist: definition → serialization (to_dict/from_dict) → aggregation → consumers → tests. The checklist is in the WO, not in the agent's head.

### 4. Parallel dispatch without post-completion verification

Launching 7 agents in parallel is efficient. Assuming all 7 completed correctly is dangerous. The parallel dispatch saved time but the verification gap (no `git diff` check before accepting results) cost time when 3 agents had to be re-executed.

**What we'd do differently:** Parallel dispatch + sequential verification. Launch agents in parallel, but collect results sequentially with `git diff` checks between each.

---

## What We'd Do Differently From Day 1

### 1. Create the Sources of Truth index before the second session

The first session can get away without it. By the second session, two agents have written documents that may conflict. The Sources of Truth index costs 10 minutes to create and prevents hours of contradiction resolution.

### 2. Enforce onboarding checklist reading order from the start

Early sessions had agents reading files in random order, consuming context on low-priority documents before reaching the critical orientation files. Defining the reading order on Day 1 would have saved ~30% of context waste across all sessions.

### 3. Build the PM briefing file immediately

The PM inbox grew to 23 files because there was no single entry point. The PM had to read everything to figure out what needed attention. A rolling briefing file (PM_BRIEFING_CURRENT.md) from Day 1 would have prevented the inbox from becoming a cognitive dump.

### 4. Treat the governance framework as a product, not a side project

The governance documents emerged reactively — each one was created after a failure. Treating them as a product from Day 1 (with versioning, testing, and deliberate design) would have reduced the number of failures needed to discover each pattern. Some failures are necessary for learning; others are preventable with upfront investment.

### 5. Research before verification

The verification pass should have consumed the research corpus first. Instead, verification ran against raw SRD text, and research cross-referencing happened after the fact. Running research first would have prevented the Design Decision Blindness failure and reduced the WRONG count by ~30% on the first pass.

### 6. Plain English debrief pass from Day 1

Early debriefs were technical-only — full of rule IDs, pipeline positions, and dataclass names. The operator could track task completion ("WO done, tests pass") but couldn't build understanding of what the system actually did or why specific changes mattered. Adding the Plain English pass (3 questions: what problem, what it does, why it matters) would have accelerated the operator's learning curve significantly. The operator lost several cycles of learning velocity before realizing the debrief format needed a translation layer.
