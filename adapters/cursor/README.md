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

Cursor doesn't have a first-class equivalent of Claude Code's
per-agent tool permissions (e.g. a mechanically-enforced read-only mode for
the Expediter). Two ways to approximate the brigade structure:

1. **Sequential, single-session:** work through Ticket → Fire → Plate →
   Expedite as explicit phases in one conversation, with the rules file
   instructing Cursor to explicitly announce which phase it's in and to
   self-enforce the boundaries of that phase (e.g. "I am now in Expedite —
   I will not edit files, only report findings against the Ticket").
   This relies on instruction-following rather than a tool-level
   permission gate, so it's weaker than the Claude Code adapter — treat
   the Expediter phase's "read-only" as a strong convention, not a
   guarantee, and have a human spot-check it periodically.
2. **Separate chats:** open a fresh Cursor chat for the Expedite phase
   specifically, with a prompt that only gives it the Ticket and the diff
   to review, not edit access framing. This gets closer to real separation
   at the cost of manually carrying context between chats.

Prefer (1) for small/solo projects, (2) when the review step matters more
(larger teams, higher-risk changes).

## Setup

See [`setup/attach.md`](../../setup/attach.md) for the full walkthrough.
The short version: symlink or submodule `tabouleh/` into the project, run
`cursorrules.template` through the project's Mise en Place to produce a
real rules file, and place it per the format guidance above.
