# Changelog

The running log of [`LINE_MEETING.md`](LINE_MEETING.md) findings — real
weaknesses found while doing real work on an attached project, and the
specific fix each one led to. Every entry answers "why does this rule
exist." Deliberate feature additions to Tabouleh (planned, not
incident-driven) aren't logged here — they don't need the "what happened"
half of the format, just normal design write-up in their own file.

Newest first.

---

## 2026-08-13 — Freshly-created `.claude/agents/` files aren't spawnable mid-session

**Triggering project:** vitals, Ticket 19 — the first live test of the
Expediter independence fix.

**What happened:** vitals never actually had its Claude Code adapter
files generated (`CLAUDE.md`, `.claude/agents/`) — only symlinked and
given a Mise en Place. Generated them mid-session specifically to spawn a
real `expediter` subagent for Ticket 19's Expedite stage. The Agent tool
call failed: `Agent type 'expediter' not found. Available agents: ...` —
the newly-written file wasn't recognized as a `subagent_type` option in
the already-running session.

**Why the gap existed:** the Claude Code adapter documented spawning
`.claude/agents/*.md` role files as subagents without ever having been
exercised against a project whose adapter files were generated *during*
the same session that then tried to use them — every prior use assumed
the adapter was already in place from a previous session.

**Fix:** didn't change the underlying mechanism (nothing to fix there —
this is a real Claude Code session-lifecycle behavior, not a bug in the
adapter's design). Documented the limitation and its workaround directly
in `adapters/claude-code/README.md`: fall back to `general-purpose` with
the role file's content given directly in the spawn prompt for the
current session (still achieves genuine context isolation), and
regenerate + start a fresh session for the real named-type version going
forward.

**Files changed:** `adapters/claude-code/README.md`.

## 2026-08-13 — Expediter reviewing its own work in the same context

**Triggering project:** vitals, discovered mid-conversation while
reviewing how Tickets 1–10 had actually been executed, not from a single
specific Ticket's failure.

**What happened:** Every Expedite stage across vitals' first ten Tickets
ran in the same conversation that had just finished Fire and Plate for
that Ticket — a role switch, not an independent review. The Expediter
therefore always had full memory of its own implementation reasoning
going into the "independent" check.

**Why the gap existed:** `.claude/agents/expediter.md` existed in the
Claude Code adapter from the start, but nothing in `THE_PASS.md` or the
adapter actually instructed spawning it as a real subagent — the mapping
was documented but never exercised, so the same-context version became
the default by omission.

**Fix:** `THE_PASS.md`'s Expedite stage and `roles/expediter.md` now state
the independence requirement directly, at the source. The Claude Code
adapter (`README.md` + `CLAUDE.md.template`) now instructs actually
calling the Agent tool with the `expediter` subagent, given only the
Ticket and the diff. The Cursor adapter's default recommendation flipped
from "same-session role-switch, fine for solo projects" to "separate
chat by default" — Cursor has no subagent primitive, so this is a weaker
approximation, documented as such rather than presented as equivalent.

**Files changed:** `core/THE_PASS.md`, `core/roles/expediter.md`,
`adapters/claude-code/README.md`, `adapters/claude-code/CLAUDE.md.template`,
`adapters/cursor/README.md`, `adapters/cursor/cursorrules.template`.
