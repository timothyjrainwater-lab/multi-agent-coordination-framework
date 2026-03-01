# Pattern: Discovery-Queue Traceability

## Problem

A research sprint produces a findings memo. An audit produces a Radar table with 12 findings. A strategy document identifies 6 recommended tools and 3 architectural changes. These artifacts are filed, reviewed, and accepted.

Six months later, nothing has been built from them. The operator asks "why are we still hand-coding data that we identified as available from an OSS source?" The PM has no answer because the research was documented but never routed. The findings memo is preserved. The WO queue never received a single item from it.

This is the research-to-queue orphan: a finding that exists in documentation but has no downstream action.

The damage:
- Work already done in research has to be rediscovered
- Approved approaches are bypassed in favor of slower manual alternatives
- Operator trust erodes when decisions don't survive into execution
- The documentation becomes noise — reading it reveals findings that were never acted on

## Solution

Every research artifact, audit output, or strategy document must have all of its action items entered into a traceability register and assigned a disposition before the artifact is considered closed.

### The Three Valid Dispositions

```
WO DISPATCHED      — a work order exists in the queue or has been accepted
DEFERRED           — explicitly parked with a rationale and a review date
CLOSED             — finding is superseded, invalid, or not actionable; reason stated
```

A finding with no disposition is not "open" — it is a traceability failure.

### The Register

Maintain a single discovery-queue traceability register. Each research artifact gets an entry:

```markdown
## [Artifact Name] — [date]
Source: [file path or link]

| Finding | Disposition | WO / Notes |
|---------|-------------|------------|
| [Finding 1] | WO DISPATCHED | WO-DATA-FEATS-001 |
| [Finding 2] | DEFERRED | Requires WeChat decryption first — revisit Q2 |
| [Finding 3] | CLOSED | Superseded by architectural change in Batch AC |
| [Finding 4] | ??? | — |     ← This row is a governance failure
```

The register is reviewed on a fixed cadence (every N batches or every PM boot). Any row with no disposition is treated as a blocker on the next research/audit dispatch — the PM must route the unrouted items before opening new research.

## Why "Documented" Is Not Enough

There is a common mistake in treating "filed" as equivalent to "actioned." A finding memo in a reviewed folder looks like work product. It is not work unless:

1. Someone read the finding, and
2. Decided what to do with it, and
3. Recorded that decision where it can be tracked

Step 3 is the failure point. Steps 1 and 2 often happen implicitly, in the moment, during a session. Without step 3, the decision evaporates between sessions. The next session starts fresh. The finding is rediscovered or, worse, bypassed without being rediscovered.

The traceability register is step 3 made mandatory.

## Implementation

### At Research Close

When a research artifact is filed, the PM must immediately populate the register entry for that artifact. This happens before the artifact is archived. A filing without a register entry is an incomplete close.

### At Audit Close

When an audit debrief is filed with a Radar table, each Radar finding gets a disposition before the audit WO can be ACCEPTED:
- CRITICAL/HIGH findings → WO dispatched immediately or explicitly escalated
- MEDIUM findings → either WO queued or DEFERRED with rationale
- LOW findings → may be CLOSED or DEFERRED on PM judgment; rationale required

The auditor files findings. The PM routes them. Both steps are required.

### At Boot (PM Seat)

On PM boot, after reading the WO queue and briefing:
- Read the traceability register
- Identify any findings with no disposition
- Route them before any other PM work

An unrouted finding is a first-class open item, not background context.

### At the Fixed Review Cadence

Every N batches (configurable — 3 and 5 batch intervals are common for the register and the backlog respectively):
- Review all entries with DEFERRED disposition
- For each: is the deferral rationale still valid? If not, promote to WO.
- For each: has the review date passed? If so, either extend explicitly or promote.

A DEFERRED finding without a review date is treated as a traceability failure.

## Relationship to Research-to-Build Pipeline

The Research-to-Build Pipeline pattern handles how to correctly convert a raw operator insight into builder-ready work orders. This pattern handles the enforcement gate that ensures the pipeline actually completes.

The pipeline can fail at any stage:
- **Burst captured but no Research WO** → tracked in the intake queue
- **Research complete but no Brick** → tracked as research-stage stall
- **Brick ready but no Builder WO** → tracked in the traceability register
- **Builder WO accepted but finding never closed** → tracked in the traceability register

Discovery-queue traceability catches the last two. The intake queue catches the first two. Together they close all the gaps in the pipeline.

## Anti-Patterns

**The findings cemetery:**
A folder full of reviewed memos, debriefs, and audit reports — all correctly filed, all with no register entries. The project "has good documentation" but the documentation hasn't moved anything. The PM reads artifacts at session start and gains context but takes no routable action from them. Artifacts inform but never dispatch.

**The implicit routing:**
"I read finding 4 and decided it wasn't important." Without recording that decision, future agents will encounter finding 4 again and have to re-decide — or worse, assume it was an oversight and treat it as open. Explicit dispositions are not bureaucracy; they are the only way to communicate past decisions to future context windows.

**The permanent defer:**
A finding DEFERRED without a review date effectively becomes closed without being declared closed. If the rationale is still valid after a year, the finding should be CLOSED with documentation of why it won't be acted on. DEFERRED is not a permanent state — it is a timed pause.
