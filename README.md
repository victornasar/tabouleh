# Tabouleh

Tabouleh is a portable, model-agnostic AI coding harness. It defines *how* an AI
agent (or a small team of them) should work on a codebase — what it plans
before it touches code, what it's forbidden from doing without asking, and
how work gets checked before it's called done — independent of which AI tool
or model you're using.

It lives in its own repo and gets **attached** to other projects. It is not
copy-pasted in. A project that wants Tabouleh symlinks or submodules this
repo in, then generates a thin, tool-specific adapter file (a `CLAUDE.md`, a
`.cursorrules`, etc.) that points back at it. Update Tabouleh once, and every
attached project can pull the update.

## Why a restaurant kitchen

Software work done by an agent has the same failure mode as a kitchen with no
structure: someone starts cooking before anyone agreed what's being made, food
goes out without anyone checking it against the order, and when something's
wrong nobody knows which step to blame. Professional kitchens solve this with
a division of labor and a checkpoint before anything leaves the pass. Tabouleh
borrows that structure because it maps cleanly onto what an AI coding agent
actually needs: a planning step, an execution step, and an independent check —
with explicit rules about what requires a human before it happens.

The restaurant terms are **labels for real mechanics**, not decoration. You
should be able to read [`core/KITCHEN_RULES.md`](core/KITCHEN_RULES.md) and
[`core/THE_PASS.md`](core/THE_PASS.md) with zero interest in the metaphor and
still know exactly what happens at each step. Here's the mapping so the
labels aren't a puzzle:

| Kitchen term | What it actually is |
|---|---|
| **Kitchen Brigade** | The set of agent roles: Executive Chef, Line Cook, Expediter |
| **Executive Chef** | The planner. Turns a request into a Ticket before any code is written. |
| **Line Cook** | The implementer. Executes exactly what's on the Ticket. |
| **Expediter** | The reviewer/QE. Read-only. Checks finished work against the Ticket before it's called done. |
| **Ticket** | The spec/task file: problem, approach, files touched, acceptance criteria, rollback plan. |
| **Mise en place** | The pre-work audit of a project: stack, conventions, risky areas, test setup. |
| **Kitchen Rules** | The non-negotiable safety rules — what needs confirmation, what's flatly forbidden, what escalates. |
| **Recipes** | Reusable step-by-step procedures for common tasks (e.g. "write a safe migration"). |
| **The Pass** | The workflow and its handoff points between roles. |
| **Walk-in** | The shared context/memory a project keeps (mise en place results, past tickets, conventions). |
| **Fire it** | Execute an approved Ticket. |

## Quick start

New to Tabouleh and want to attach it to a project right now? Go straight to
[`setup/attach.md`](setup/attach.md) — it's a step-by-step walkthrough with a
copyable prompt.

## Repo map

```
tabouleh/
  core/
    KITCHEN_RULES.md        Safety rules: confirm / block / escalate, by action
    THE_PASS.md              The workflow: ticket -> fire -> plate -> expedite -> serve
    PARALLEL_LINE.md          Opt-in: running multiple Tickets at once
    LINE_MEETING.md           How a real weakness becomes a fix to core/
    CHANGELOG.md              Log of every Line Meeting finding and its fix
    roles/                   One file per brigade role
    recipes/                 Reusable procedures for common tasks
    templates/                Blank Ticket and Mise en Place formats
  adapters/
    claude-code/             How Tabouleh maps into CLAUDE.md + .claude/agents/
    cursor/                  How Tabouleh maps into .cursorrules
  setup/
    attach.md                 How to wire Tabouleh into a new project
```

## Reading order

If you're evaluating Tabouleh or onboarding to a project that uses it, read
in this order:

1. This README (you're here)
2. [`core/KITCHEN_RULES.md`](core/KITCHEN_RULES.md) — the rules that can't be
   negotiated away regardless of task
3. [`core/THE_PASS.md`](core/THE_PASS.md) — the workflow every piece of work
   goes through
4. [`core/roles/`](core/roles/) — what each role does and doesn't do
5. Whichever adapter matches your tool: [`adapters/claude-code/`](adapters/claude-code/)
   or [`adapters/cursor/`](adapters/cursor/)

## Scope

Tabouleh is a template/toolkit repo. It has no "production" deployment of its
own, but its `core/` files are load-bearing for every project that attaches
it — so changes here follow the same discipline described in
[`core/KITCHEN_RULES.md`](core/KITCHEN_RULES.md), including confirmation
before deleting or rewriting anything in `core/` or `adapters/`. When a real
weakness in the harness itself turns up while working on an attached
project, [`core/LINE_MEETING.md`](core/LINE_MEETING.md) is the actual
procedure for proposing and logging the fix — not an ad hoc conversation
each time.
