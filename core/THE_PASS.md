# The Pass

The Pass is the workflow every piece of work moves through, and the set of
handoff points between roles. Nothing moves to the next stage without the
current stage's exit criteria being met. This document defines those
criteria concretely — the kitchen terms label the stages, they don't hide
what happens at each one.

```
  TICKET  --->  FIRE  --->  PLATE  --->  EXPEDITE  --->  SERVE
 (Chef)      (Line Cook)  (Line Cook)   (Expediter)     (done)
```

This describes one Ticket at a time — the default, and almost always the
right choice. When there's an actual backlog of independent, already-
approved Tickets, see [`PARALLEL_LINE.md`](PARALLEL_LINE.md) for running
several of these pipelines at once; it adds claiming, isolation, and a
merge-back step around the same five stages below, unchanged.

## Stage 1 — Ticket

**Owner:** Executive Chef.
**What happens:** The Chef turns a raw request into a Ticket using
[`ticket.template.md`](templates/ticket.template.md): Problem, Approach,
Files Touched, Acceptance Criteria (a checklist, not prose), Rollback Plan.
If the project hasn't had Mise en Place run yet, or the request touches an
area Mise en Place didn't cover, that gets done or updated first.

**Exit criteria (must all be true before moving to Fire):**
- The Ticket has concrete, checkable acceptance criteria — not "works
  correctly" but specific, verifiable statements.
- Files Touched is a real list, not "TBD."
- A rollback plan is stated (see [`KITCHEN_RULES.md`](KITCHEN_RULES.md) §7).
- Nothing in the Ticket requires an action in Kitchen Rules' BLOCK table.
  If it does, the Ticket is rewritten to avoid it, or the request is
  escalated to the human before a Ticket is even written.
- The human has approved the Ticket. **No code is written before this.**

## Stage 2 — Fire

**Owner:** Line Cook.
**What happens:** "Fire it" = execute the approved Ticket. The Line Cook
implements exactly what the Ticket describes, touching only the files it
lists, within the tool/permission boundaries in
[`line-cook.md`](roles/line-cook.md).

**Exit criteria (must all be true before moving to Plate):**
- Every item in the Ticket's approach has been implemented.
- No files were touched outside the Ticket's Files Touched list. If that
  turned out to be necessary, this is scope creep — see Rule 8 in
  [`KITCHEN_RULES.md`](KITCHEN_RULES.md) — and gets flagged rather than
  silently done.
- Any action in Kitchen Rules' CONFIRM table encountered mid-implementation
  was actually confirmed before it happened, not after.

## Stage 3 — Plate

**Owner:** Line Cook (self-review, before handing off).
**What happens:** The Line Cook checks its own work against the Ticket
before anyone else looks at it — the equivalent of a cook checking a dish
before it goes to the pass, not after. Follow
[`recipe-self-review.md`](recipes/recipe-self-review.md).

**Exit criteria (must all be true before moving to Expedite):**
- Every acceptance criterion on the Ticket has been checked, one by one,
  against the actual result — not assumed.
- Tests relevant to the change exist and pass locally.
- No debug output, commented-out code, or scratch artifacts left in the
  diff.
- The diff matches the Files Touched list exactly.
- The rollback plan from the Ticket is still accurate for what was
  actually built (if the implementation diverged from the plan, the
  rollback plan is updated to match).

## Stage 4 — Expedite

**Owner:** Expediter.
**What happens:** The Expediter — with **read-only** access, it cannot edit
anything — checks the plated work against the Ticket's acceptance criteria
one item at a time. This is an independent check, not a rubber stamp of the
Line Cook's self-review. See [`expediter.md`](roles/expediter.md) for the
full checklist.

**The Expediter is invoked as a genuinely separate context, not a role
switch.** When the tool in use supports spawning an independent agent
(e.g. Claude Code's Agent/Task tool with a dedicated `expediter`
definition), Expedite means actually spawning it — handing it only the
Ticket and the diff, not the Fire/Plate conversation that produced them.
A reviewer who remembers writing the code will defend its own reasoning
instead of checking it; a reviewer with no memory of writing it can't.
When the tool has no such mechanism, the adapter for that tool states its
best available approximation explicitly rather than silently treating a
same-context role-switch as equivalent — see each adapter's own notes on
this (Claude Code's is closest to the real thing; Cursor's is a weaker
approximation, documented as such).

**Possible outcomes:**

1. **Pass** — every acceptance criterion is met, no Kitchen Rules
   violations, diff matches scope. Moves to Serve.
2. **Send-back** — one or more acceptance criteria aren't met, or the diff
   has scope creep, or tests are missing/failing. The Expediter writes
   specific, itemized feedback (which criterion, why it's not met — not
   "doesn't look right") and returns the work to the Line Cook. This goes
   back to Stage 2 (Fire) with that feedback attached.
3. **Escalate** — a Kitchen Rules violation was found (see
   [`KITCHEN_RULES.md`](KITCHEN_RULES.md) §8), the Ticket itself turns out
   to be wrong or ambiguous in a way no amount of re-implementation fixes,
   or the retry limit below has been hit. Goes to the human, not back to
   the Line Cook.

### Loopback policy (send-back retry limit)

- A send-back counts as one loop. **After 2 send-backs on the same Ticket
  (3 total attempts), the Expediter escalates to the human instead of
  sending back a 3rd time**, even if the remaining issues look minor.
- The reasoning: two failed attempts at the same Ticket usually means the
  Ticket itself is underspecified or the approach is wrong, not that the
  Line Cook needs one more try. That's a planning problem, which routes
  back to the Executive Chef via the human, not another implementation
  cycle.
- Each send-back's feedback is cumulative context for the next attempt —
  the Line Cook sees what previous rounds got flagged for, so the same
  issue doesn't get reintroduced.
- **Re-verification on attempt 2+ scopes to what changed, not everything
  again.** Attempt 1's Expedite already established the full picture. If
  attempt 2 only touched what the send-back named, re-confirm the fix,
  re-run whatever a full check would need to re-run to trust the result
  (a build, the specific test), and reconcile the Ticket's own bookkeeping
  (Files Touched, acceptance criteria) against the now-current diff — but
  don't re-derive the whole investigation from zero each round. Full-cost
  re-verification on every attempt turns a cheap fix into an expensive
  loop for no added rigor. See [`expediter.md`](roles/expediter.md)'s
  "Keep the report proportional."

## Stage 5 — Serve

**What happens:** Work is done. This means: acceptance criteria verifiably
met, tests passing, no unresolved Expediter feedback, and — per
[`KITCHEN_RULES.md`](KITCHEN_RULES.md) — any CONFIRM-gated action along the
way was actually confirmed, not skipped. "Serve" is the state where a human
can merge/deploy without re-checking the Expediter's work from scratch.

**What "served" does not mean:** it does not mean deployed to production.
Deployment is its own CONFIRM-gated action (Kitchen Rules §6) that happens
after Serve, at the human's direction.

**Serve is also a natural session boundary.** Once a Ticket (or a batch of
related Tickets) is Served, the conversation that produced it has done its
job — carrying its full history into unrelated work that follows costs
context and money for no benefit the next piece of work actually needs.
Starting a fresh session for the next distinct chunk of work, rather than
extending an already-long one indefinitely, is the default, not something
to only consider once a session is struggling under its own size.

## Summary table

| Stage | Owner | Produces | Exit gate |
|---|---|---|---|
| Ticket | Executive Chef | Approved Ticket | Human approval |
| Fire | Line Cook | Implementation | Matches Ticket scope |
| Plate | Line Cook | Self-reviewed diff | Self-checklist passed |
| Expedite | Expediter | Pass / Send-back / Escalate | Acceptance criteria independently verified |
| Serve | — | Done | No open Expediter feedback |
