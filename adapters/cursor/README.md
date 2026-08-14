# Adapter: Cursor

How Tabouleh's core concepts map into Cursor's rules mechanism.

## Mapping

| Tabouleh concept | Cursor mechanism |
|---|---|
| `core/KITCHEN_RULES.md` + `core/THE_PASS.md` | Content inlined into `.cursor/rules/tabouleh.mdc` (or a legacy `.cursorrules` file — see below), applied project-wide |
| Executive Chef, Line Cook, Expediter | No native sub-agent/role separation in Cursor the way Claude Code's `.claude/agents/` provides it — see "Role separation" below for how to approximate it |
| Ticket | A markdown file (e.g. `tickets/<slug>.md`), created from `core/templates/ticket.template.md`, referenced in chat when starting work |
| Mise en Place | A markdown file (e.g. `MISE_EN_PLACE.md` at project root), included as an `@file` reference or inlined into the rules file |
| Walk-in | Cursor's own project context plus the rules file — Tabouleh doesn't add new machinery here |
| Recipes | Referenced by relative path into the attached `tabouleh/` repo from the rules file or pulled in via `@file` when relevant |

## Rules file format

Cursor supports both the newer `.cursor/rules/*.mdc` format (preferred —
supports multiple scoped rule files with frontmatter controlling when they
apply) and the older single `.cursorrules` file. Use
[`cursorrules.template`](cursorrules.template) either way:

- **Preferred:** save it as `.cursor/rules/tabouleh.mdc` with `alwaysApply:
  true` in the frontmatter, so Kitchen Rules and The Pass are always in
  context regardless of what file is open.
- **Legacy:** save it as `.cursorrules` at the project root if the project
  hasn't migrated to the newer format.

## Role separation

Cursor doesn't have a first-class equivalent of Claude Code's per-agent
tool permissions or a subagent primitive to spawn — there's no mechanism
here that gets as close to real independence as the Claude Code adapter's
Agent-tool invocation does. Two ways to approximate the brigade structure,
in preference order:

1. **Separate chats (default).** Open a fresh Cursor chat for the Expedite
   phase, and give it *only* the Ticket file and the diff to review —
   don't carry the Fire/Plate conversation into it, and don't frame it
   with edit access. This is the closest approximation available in
   Cursor to a context that never saw the implementation reasoning, which
   is the actual thing that makes a review independent — not just "asked
   to check its own work" phrased as a separate step. Default to this even
   for solo, low-stakes projects; the fresh-context benefit doesn't
   disappear just because no one else is reviewing alongside you.
2. **Sequential, single-session** (fallback for trivial Tickets only):
   work through Ticket → Fire → Plate → Expedite as explicit phases in one
   conversation, with the rules file instructing Cursor to announce which
   phase it's in and self-enforce that phase's boundaries (e.g. "I am now
   in Expedite — I will not edit files, only report findings against the
   Ticket"). This relies entirely on instruction-following, not a
   tool-level gate or a fresh context — treat its "read-only" and
   "independent" both as conventions, not guarantees, and don't reach for
   it just to save the cost of opening a second chat.

## Setup

See [`setup/attach.md`](../../setup/attach.md) for the full walkthrough.
The short version: symlink or submodule `tabouleh/` into the project, run
`cursorrules.template` through the project's Mise en Place to produce a
real rules file, and place it per the format guidance above.
