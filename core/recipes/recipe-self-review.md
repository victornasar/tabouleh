# Recipe: Self-Review (Plate)

Owner: Line Cook. Run this before handing work to the Expediter — it's the
"Plate" stage in [`THE_PASS.md`](../THE_PASS.md). The goal is to catch what
the Expediter would catch, so the Expediter's job is confirmation, not
discovery.

## Steps

1. **Re-read the Ticket, not your memory of it.** Open the actual Ticket
   document. Implementation tends to drift from plan in small ways; re-read
   rather than relying on recall of what you set out to do.

2. **Walk the acceptance criteria one by one, and actually check each
   one.** For each checklist item: run the relevant test, exercise the
   behavior, or read the specific code path — don't mark it done because
   the surrounding code "looks like it would handle that." If a criterion
   can't be checked without a manual step, do the manual step now, not
   assume the Expediter will.

3. **Diff against Files Touched.** Run `git diff --stat` (or equivalent)
   and compare the file list against the Ticket's Files Touched. Anything
   extra is scope creep — either justify it explicitly in the handoff
   notes or revert it before handing off.

4. **Run the full relevant test suite, not just the new tests.** New tests
   passing doesn't confirm nothing else broke. Run the tests scoped to the
   area touched at minimum; run the full suite if the project's size makes
   that practical.

5. **Scan the diff for leftovers.** Debug prints, commented-out old code,
   `TODO` markers left as a substitute for actually finishing something,
   hardcoded values that were meant to be temporary, unused imports.

6. **Check secrets and config didn't leak in.** No credential values,
   tokens, or `.env` contents anywhere in the diff — see
   [`KITCHEN_RULES.md`](../KITCHEN_RULES.md) §5.

7. **Verify the rollback plan is still accurate.** If implementation
   diverged from the Ticket's original approach (e.g. touched one more
   file than planned, or a migration ended up structured differently),
   update the rollback plan to match what was actually built, not what was
   originally planned.

8. **Write the handoff notes.** For the Expediter: which acceptance
   criteria were checked and how, test results, and any deviations from
   the original Ticket with a one-line justification for each. This isn't
   busywork — it's what lets the Expediter verify efficiently instead of
   re-deriving context from scratch.

## Self-review checklist (copy into handoff notes)

- [ ] Re-read the Ticket in full.
- [ ] Every acceptance criterion checked individually against actual
      behavior/test output.
- [ ] Diff matches Files Touched exactly (or deviations are justified in
      notes).
- [ ] Relevant test suite run and passing.
- [ ] No debug artifacts, dead code, or leftover TODOs standing in for
      unfinished work.
- [ ] No secret values anywhere in the diff.
- [ ] Rollback plan updated to match what was actually built.
- [ ] Handoff notes written for the Expediter.
