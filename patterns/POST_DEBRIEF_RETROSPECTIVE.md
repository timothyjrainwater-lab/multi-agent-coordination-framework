# Pattern: Post-Debrief Retrospective

## Problem

The formal debrief format — what was changed, what tests pass, what was found — is oriented toward delivery. It answers the question: *"Did the builder do what the WO said?"*

That's the right question for the delivery record. But it's the wrong question for surfacing what the builder noticed that wasn't in scope.

**Builders are uniquely positioned to see things nobody else can see.** They're inside the code with full context for the specific subsystem they worked on. They see the seams between their WO and adjacent code. They notice coupling risks, naming drift, patterns that are about to repeat, corners that look like they're about to cause a problem.

None of this fits naturally into the three-pass debrief format. The debrief is about delivery. The builder's peripheral observations aren't about delivery — they're about what's coming.

**The failure mode:** The builder completes the WO, writes an accurate debrief, and closes the session. Two sessions later, a related WO runs into the coupling risk the previous builder noticed but didn't mention. The risk was real, observable, and available — but it lived in the builder's context window and died when the session ended.

## Solution

### Rule

**After every accepted debrief, ask the builder: "Anything else you noticed outside the debrief?"**

This question is mandatory. It is not optional follow-up. It doesn't depend on whether the builder seems to have more to say. The question changes what the builder thinks about — it activates a different lens than the delivery-focused debrief.

The question fires after the formal debrief is filed and reviewed, not during. The timing matters: during delivery, the builder is focused on delivery. After the formal record is complete and the pressure is off, peripheral observations become accessible.

### What This Question Surfaces

| Category | Example |
|----------|---------|
| Coupling risks | "That shared utility function is called from 6 other places — if it changes shape, they all break" |
| Naming drift | "There are two functions with similar names that do different things — this is going to confuse the next builder" |
| Silent gaps | "The error case is handled, but the warning case returns quietly with no log entry" |
| Structural risks | "The test passes but it's testing against a hardcoded fixture that will be wrong if the schema changes" |
| Upcoming collision | "The next WO in the queue is going to hit a conflict with what I just changed — worth flagging" |
| Observations outside scope | "I noticed while reading the surrounding code that [X] seems incorrect — didn't touch it, but worth knowing" |

### Filing Findings

Any signal that emerges from this question gets filed immediately as a FINDING:

```markdown
FINDING-[SUBSYSTEM]-[SEQUENCE]-001
Severity: LOW | MEDIUM | HIGH
Source: Post-debrief retrospective — WO-[ID]
Summary: [One sentence]
Detail: [What the builder observed]
Recommended action: [Queue for PM triage | Investigate before next batch | Block next dispatch]
```

The finding is filed before the session closes. Not "noted for later." Not "mentioned in the debrief." Filed as a finding that the PM will see and triage.

### PM Enforcement

The PM must ask this question after every debrief acceptance. The PM does not ask "is there anything else?" in a way that signals the right answer is "no." The question is active and specific:

*"Anything else you noticed outside the debrief? Coupling risks, naming drift, anything that caught your eye while you were in there?"*

The specificity matters. "Anything else?" reads as "are we done?" "Anything else you noticed... coupling risks, naming drift..." reads as "I'm genuinely asking for peripheral observations."

If the builder has nothing: record "Post-debrief retrospective: no additional findings." One line. The record shows the question was asked.

### Relationship to the Three-Pass Debrief

The three-pass debrief has a Pass 3 retrospective. The post-debrief question is **separate** from that retrospective, not a repeat of it.

- **Pass 3 retrospective:** Covers what happened during the WO — drift caught, patterns used, lessons for next time. Oriented toward *this* session.
- **Post-debrief question:** Covers what the builder noticed that has nothing to do with this WO. Oriented toward *future* sessions and adjacent risks.

Both are required. Pass 3 doesn't substitute for the question; the question doesn't substitute for Pass 3.

## Implementation

### When to Ask

| Timing | When | Notes |
|--------|------|-------|
| **After debrief review, before closing the session** | PM reads the debrief, confirms it's acceptable, then asks | Timing ensures the builder is still in context |
| **Not during implementation** | Never ask mid-session | Splits builder focus |
| **Not before the debrief** | Asking "anything to add?" before Pass 3 is written conflates the two |

### How to Record

Add a section to the PM's session record:

```markdown
## Post-Debrief Retrospective — WO-[ID]

Asked: [date]
Builder response:
- [Finding 1 — filed as FINDING-...]
- [Finding 2 — filed as FINDING-...]
- No additional findings
```

### Integration with Findings Triage

Findings from post-debrief retrospectives enter the same triage queue as findings from audits and other sources. The PM triages them at the next queue review. They have equal standing with audit findings — they came from a different observation angle but they're just as valid.

## When to Use

- After every accepted builder debrief
- No exceptions for "short" WOs or "routine" fixes
- Even when the builder's debrief is comprehensive and seems complete

## When NOT to Use

- During the WO (don't interrupt delivery with scope-expansion questions)
- As a substitute for an audit (this catches peripheral builder observations, not systematic subsystem analysis)

## Anti-Patterns

- **Asking "are we done?"** — That's not the question. The question is "did you notice anything?"
- **Accepting "no" too quickly** — A gentle follow-up is fine: "Nothing about the surrounding code? Any naming that looked off?"
- **Not filing the findings** — "I'll remember it for next time" is not filing. If it's real, it gets a FINDING entry today.
- **Batching up findings** — File before closing the session. Findings that aren't filed in the session often don't get filed at all.

## Real Example

A builder completing a work order on a skill modifier system noticed — while reading the code to understand the surrounding context — that a utility function used by multiple callers had two different parameter ordering conventions. The debrief covered only the WO scope. The peripheral observation wasn't formally part of the WO.

The post-debrief question surfaced it. The builder described: *"The skill name normalization function has two call signatures in different parts of the codebase — one passes skill name first, the other passes it second. I didn't touch it but it looks like a latent bug waiting to trigger."*

That observation was filed as a FINDING. Two batches later, a different WO was dispatched to normalize the call signature and add a test that would have failed against the bad ordering. The bug never reached production.

**Without the question:** The observation would have died in the session context window.
**With the question:** It became a filed finding, a dispatched WO, and a prevented regression.
