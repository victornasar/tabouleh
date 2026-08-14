# The Parallel Line

The Pass, as written, assumes one Ticket moves through Ticket → Fire →
Plate → Expedite → Serve at a time. That's still the default — most work
should just use it. The Parallel Line is the opt-in extension for when
there's an actual backlog of independent, already-approved Tickets ready
to fire at once, and it names the real machinery that requires: claiming,
isolation, and a merge-back step The Pass doesn't otherwise have.

This document doesn't replace [`THE_PASS.md`](THE_PASS.md) — every stage
it defines still applies per-Ticket, unchanged. This is what sits around
multiple instances of it running at once.

## When to use this

Only when all of the following are true:

- There are **two or more Tickets already approved** (Stage 1 done for
  each, independently) — the Parallel Line does not change how a Ticket
  gets approved, only how approved Tickets get executed.
- Their **Files Touched lists are pairwise disjoint** — no file appears in
  more than one of the Tickets being run concurrently (see the rule
  below).
- There's a real reason to want the speed — a backlog waiting, not
  "might as well." Reaching for this on one or two Tickets that could
  just run sequentially adds coordination cost for no benefit.

If any of these isn't true, run the Tickets one at a time through the
normal Pass instead.

## The core rule: disjoint Files Touched

**No two Tickets fired in parallel may share a file in their Files
Touched lists — not even one.** This is the single invariant everything
else here depends on. Two Line Cooks editing the same file at the same
time in different worktrees is exactly the collision Kitchen Rules exists
to prevent, and it will surface as either a silent overwrite or a merge
conflict that looks like a tooling problem but is actually a process
violation.

Checking this is part of claiming (below), not something to verify after
the fact. If a Ticket's approach turns out to need a file already claimed
by another in-flight parallel Ticket, that Ticket doesn't fire yet — it
waits for the other to reach Serve, or goes back to the Executive Chef to
be re-scoped.

## Claiming a Ticket

Before a Line Cook fires a Ticket under the Parallel Line, it claims it:

1. Confirm the Ticket's `Status` is `approved` and it has no
   `Worktree` value set (see the updated
   [`ticket.template.md`](templates/ticket.template.md) metadata).
2. Set `Status: in-progress` and `Worktree: <branch-name>` (see naming
   below), and commit that change to the Ticket file on the shared
   integration branch (`main`, or whatever the project treats as
   canonical) immediately — push it before doing anything else. This is
   the claim.
3. Only after the claim is pushed does Fire actually start.

**Known limitation, stated honestly rather than hidden:** two Line Cooks
claiming at nearly the same instant can still race — this isn't a
database with real locking, it's a markdown file and a git push. The
mitigation is that whichever claim commit reaches the integration branch
first wins; a Line Cook whose claim gets superseded (its push rejected, or
it pulls and sees a different `Worktree` value already recorded when it
expected none) aborts and either waits or picks a different Ticket. This
is a low-probability race in practice — a human is the one deciding to
fire multiple Tickets, not the harness spontaneously starting them — but
it's a real gap, not a solved one.

## Isolation: one worktree, one branch per Ticket

Each claimed Ticket gets its own git worktree and branch, not just a
mental note to "be careful":

```bash
git worktree add ../<repo>-ticket-<N> -b ticket/<N>-<slug>
```

The Line Cook does all Fire and Plate work exclusively inside that
worktree. It never touches the main working tree or another Ticket's
worktree. This is what actually enforces the disjoint-files rule
mechanically instead of just by discipline — even if two Tickets somehow
did overlap, they'd be editing separate checkouts, and the collision would
surface as a merge conflict at merge-back (see below) rather than a
silent overwrite.

## Fire, Plate, Expedite — unchanged, just per-branch

Nothing about the stages themselves changes. Fire and Plate happen inside
the Ticket's worktree exactly as [`THE_PASS.md`](THE_PASS.md) describes.
Expedite happens the same way too — and per the Expediter's independence
requirement (see `THE_PASS.md`'s Expedite stage and
[`roles/expediter.md`](roles/expediter.md)), it should still run as a
separate agent invocation wherever the tool supports it, just pointed at
that Ticket's branch diff specifically rather than the main working tree.
A send-back under the Parallel Line goes back to Fire in the *same*
worktree — it doesn't reclaim or re-isolate.

## Merge-back (new stage, between Expedite and Serve)

A Ticket that passes Expedite under the Parallel Line isn't Served yet —
it has to get back to the integration branch first:

1. Merge (or rebase, per the project's own convention) the Ticket's
   branch into the integration branch.
2. **A merge conflict at this step is not a tooling problem to resolve
   quietly — it's a signal the disjoint-files invariant was violated**
   (scope crept past Files Touched, or two Tickets' approaches turned out
   to overlap in a way that wasn't caught at claim time). Escalate it the
   same way a Kitchen Rules violation would be escalated, rather than
   force-resolving and moving on.
3. On a clean merge: remove the worktree (`git worktree remove`), delete
   the Ticket's branch, and set `Status: done`. *Now* it's Served.

## Kitchen Rules additions specific to parallel execution

- Firing two Tickets whose Files Touched lists overlap, even partially,
  is **BLOCK** — not a judgment call, per the core rule above.
- Running Tickets in parallel does not bundle or reduce the human's
  confirmation obligations. If two parallel Tickets both hit a
  CONFIRM-gated action around the same time, each gets its own explicit
  human response — a yes to one is not a yes to the other, and the human
  should expect to be interrupted by both, not a merged summary of both.

## Tool support

- **Claude Code:** the real version — `git worktree` plus the Agent tool
  spawning a `line-cook` subagent per claimed Ticket, each pointed at its
  own worktree path. See
  [`adapters/claude-code/README.md`](../adapters/claude-code/README.md).
- **Cursor:** no subagent primitive to spawn into (the same gap as the
  Expediter fix ran into) — parallel Tickets here mean genuinely separate
  Cursor windows or sessions, one per worktree, manually opened and
  coordinated by the human. Weaker than the Claude Code version, honestly
  labeled as such rather than papered over. See
  [`adapters/cursor/README.md`](../adapters/cursor/README.md).

## Worked example

Two approved Tickets: #14 ("Settings: add a units-preference toggle,"
Files Touched: `SettingsView.swift`) and #15 ("Gym: add a weekly-reminder
notification," Files Touched: `GymView.swift`, `NotificationScheduler.swift`
(new)). Disjoint — no shared files. Both claimed: `ticket/14-units-toggle`
and `ticket/15-weekly-reminder`, each in its own worktree. Both Fire in
parallel. #15 finishes Plate first, passes Expedite, merges to `main`,
worktree removed. #14 finishes shortly after, passes Expedite, merges to
`main` (now including #15's merged change — a normal rebase/merge, no
conflict, since the files never overlapped), worktree removed. Both
`done`. Total wall-clock time: roughly however long the slower of the two
took, not both added together.
