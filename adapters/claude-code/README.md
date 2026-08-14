# Adapter: Claude Code

How Tabouleh's core concepts map into Claude Code's actual mechanisms.

## Mapping

| Tabouleh concept | Claude Code mechanism |
|---|---|
| `core/KITCHEN_RULES.md` + `core/THE_PASS.md` | Pulled into the project's `CLAUDE.md`, read at the start of every session |
| Executive Chef, Line Cook, Expediter | Three files under `.claude/agents/`, one per role |
| Ticket | A markdown file (e.g. `tickets/<slug>.md`), created from `core/templates/ticket.template.md` |
| Mise en Place | A markdown file (e.g. `MISE_EN_PLACE.md` at project root), created from `core/templates/mise-en-place.template.md` |
| Walk-in | The project's `CLAUDE.md` plus whatever memory files the project already keeps — Tabouleh doesn't introduce a separate mechanism, it just makes sure Mise en Place and prior Tickets are part of what gets loaded |
| Recipes | Referenced directly from `.claude/agents/` role files or `CLAUDE.md` by relative path into the attached `tabouleh/` repo — not copied |

## CLAUDE.md

Generated from [`CLAUDE.md.template`](CLAUDE.md.template): it references
(or inlines, if the project prefers a self-contained file) Tabouleh's
`KITCHEN_RULES.md` and `THE_PASS.md`, then appends the project's own Mise
en Place. This is the file Claude Code reads automatically, so it's the
anchor point for the whole system in this tool.

## `.claude/agents/`

Each brigade role becomes an agent definition:

- `.claude/agents/executive-chef.md` — sourced from
  `tabouleh/core/roles/executive-chef.md`, with Claude Code's agent
  frontmatter (name, description, and **no write tools** — this role plans,
  it doesn't implement) added on top.
- `.claude/agents/line-cook.md` — sourced from
  `tabouleh/core/roles/line-cook.md`, with full read/write/execute tools
  scoped to the working directory.
- `.claude/agents/expediter.md` — sourced from
  `tabouleh/core/roles/expediter.md`, with **read-only tools** in the
  frontmatter (no `Edit`, `Write`, or any tool that mutates files). This is
  where the Expediter's read-only requirement gets enforced mechanically,
  not just by convention — Claude Code's permission system is the actual
  gate.

**These are meant to be invoked as real subagents, not just documented as
a mapping.** At Expedite (see `THE_PASS.md`), the session that just ran
Fire/Plate must actually call the Agent tool with the `expediter`
subagent — e.g. "use the Agent tool to launch the `expediter` subagent,
passing it the path to the Ticket and the diff against its baseline
commit, and nothing else about how the implementation was reached." Do
**not** treat "I'll now act as Expediter" in the same conversation as
equivalent — that's the same-context weakness this mapping exists to
avoid, and Claude Code's Agent tool is specifically what makes the real
version possible here. The Executive Chef step can be run the same way
for a large or ambiguous request (spawn `executive-chef` to produce a
Ticket for review before any implementation session even starts), though
it's less critical there since the human approves the Ticket regardless.

## Parallel Line

For firing multiple approved, Files-Touched-disjoint Tickets at once (see
[`PARALLEL_LINE.md`](../../core/PARALLEL_LINE.md)) — this is Claude Code's
real mechanism for it, not an approximation:

1. Verify the disjoint-files rule yourself before claiming anything —
   don't rely on each Line Cook to notice a conflict after the fact.
2. Create one `git worktree` per Ticket being fired, per
   `PARALLEL_LINE.md`'s naming convention.
3. Spawn one `line-cook` subagent per Ticket via the Agent tool, each
   given its own worktree path and told to work exclusively within it —
   in a tool that supports backgrounding subagent calls, run them
   backgrounded so they actually proceed concurrently rather than one
   blocking the next.
4. As each finishes Plate, spawn its `expediter` subagent the same way
   Expedite normally works (see above), pointed at that Ticket's branch
   diff specifically.
5. On a pass, merge that Ticket's branch back per `PARALLEL_LINE.md`'s
   merge-back step, then remove its worktree.

## Confirmation gates

Kitchen Rules' CONFIRM actions map onto Claude Code's permission-prompt
behavior: commands and tools that would otherwise require a manual
approval in Claude Code (destructive bash commands, git push, etc.) are
exactly the category Kitchen Rules asks the agent to pause on regardless.
The adapter doesn't need to invent new machinery for this — it aligns
Kitchen Rules' language with the tool's existing prompts so an approval in
Claude Code corresponds to an approval Kitchen Rules would also have
required.

## Setup

See [`setup/attach.md`](../../setup/attach.md) for the full walkthrough.
The short version: symlink or submodule `tabouleh/` into the project, run
`CLAUDE.md.template` through the project's Mise en Place to produce a real
`CLAUDE.md`, and create the three files under `.claude/agents/` referencing
the role files above.
