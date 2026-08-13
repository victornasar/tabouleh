# Ticket: <short title>

<!--
Filled out by the Executive Chef before any code is written. See
core/recipes/recipe-ticket-writing.md for guidance and a worked example.
Do not remove sections; write "N/A" with a one-line reason if genuinely
not applicable.
-->

**Status:** draft | approved | in-progress | in-review | send-back | done
**Owner (Line Cook):**
**Retry count:** 0
**Related project Mise en Place:** <link, if applicable>

## Problem

<What's actually wrong or missing — not a restatement of the request. One
or two sentences a reader unfamiliar with the request would understand.>

## Approach

<Numbered steps, concrete enough that a Line Cook can follow them without
re-deriving the design. Name specific functions/modules/endpoints where
known.>

1.
2.
3.

## Files touched

<Real file paths, not directory-level guesses.>

-

## Acceptance criteria

<Checkable statements, not descriptions. Each one should have a clear
yes/no answer once checked.>

- [ ]
- [ ]
- [ ]

## Rollback plan

<How this gets undone if it needs to be. "Revert the commit" is sufficient
for most simple code changes — say so explicitly. Anything touching
Kitchen Rules categories (migrations, dependencies, production, deletions)
needs a specific plan — see core/recipes/recipe-safe-migration.md for the
migration case.>

## Kitchen Rules check

<Confirm the approach above doesn't require anything in KITCHEN_RULES.md's
BLOCK table. Note anything that will need a CONFIRM step during
implementation, so it isn't a surprise mid-Ticket.>

## Notes for the Expediter

<Filled in during Plate, not now — left blank at Ticket-approval time.>
