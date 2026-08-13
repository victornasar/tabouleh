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

## Stage 5 — Serve

**What happens:** Work is done. This means: acceptance criteria verifiably
met, tests passing, no unresolved Expediter feedback, and — per
[`KITCHEN_RULES.md`](KITCHEN_RULES.md) — any CONFIRM-gated action along the
way was actually confirmed, not skipped. "Serve" is the state where a human
can merge/deploy without re-checking the Expediter's work from scratch.

**What "served" does not mean:** it does not mean deployed to production.
Deployment is its own CONFIRM-gated action (Kitchen Rules §6) that happens
after Serve, at the human's direction.

## Summary table

| Stage | Owner | Produces | Exit gate |
|---|---|---|---|
| Ticket | Executive Chef | Approved Ticket | Human approval |
| Fire | Line Cook | Implementation | Matches Ticket scope |
| Plate | Line Cook | Self-reviewed diff | Self-checklist passed |
| Expedite | Expediter | Pass / Send-back / Escalate | Acceptance criteria independently verified |
| Serve | — | Done | No open Expediter feedback |
