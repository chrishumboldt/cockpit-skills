---
name: implement
description: The Implement step of a Cockpit run. Builds one ticket test-first against spec.md, addresses every review finding when a review asked for changes, runs only the focused tests, commits, and writes build.md.
disable-model-invocation: true
---

# Implement

You run the Implement step for one ticket of a Cockpit run. Build that ticket, test-first, on the branch you are on, and say what you did in `build.md`. You ask nothing: where the ticket and spec are silent, take the reading that fits them best and record it in `build.md`.

## 1. Read

- The ticket file the prompt names (`tickets/NN-<slug>.md` in the run folder): its What to build, acceptance criteria and Verified by are your scope. Say its number and title in your first message.
- `spec.md` for the decisions and terms the ticket rests on.
- Any `review-*.md` beside them: when one reports `CHANGES`, this is a fix round (step 4).
- The code you will change, the glossary and the ADRs in that area.

Stay inside the ticket. Work another ticket needs is that ticket's.

## 2. Build it test-first

Work the **red → green** loop at the seams the spec's Testing section names, one behaviour at a time: a failing test, then the least code that passes it, then the next. [tdd.md](tdd.md) is the reference for the loop, what a good test is and where mocks belong; read it before the first test.

When the slice is green, tidy what you wrote (names, duplication, a function grown long) with the tests still passing.

## 3. Verify, focused

Run the type checker, then only the tests for what you touched: the test files you wrote or changed, the tests the runner relates to the files you changed (`vitest related <files>`), and the end-to-end specs that cover the behaviour. Never run a whole suite. Cap the workers on anything that fans out: `VITEST_MAX_WORKERS=4`, Playwright `--workers=2`. Every acceptance criterion is checked by a test or by hand before you finish.

## 4. A fix round

When a review asked for changes, address **every** finding under its Changes: make the change, or, when a finding is wrong, leave the code and say why in `build.md`. Optional findings are your call; note each one you skip. Re-run the focused tests after the fixes.

## 5. Commit

Commit the work to the current branch, in one or a few commits whose messages say what changed and name the ticket ("Pause the run when a step fails (ticket 03)"). Stay on this branch, and leave pushing and merging to the run.

## 6. Write `build.md`

The path the prompt gives:

```markdown
# Build · ticket NN: <Title>

## Built
What now works, in a few lines, and the main parts changed.

## Verified
Each command run and its result (`VITEST_MAX_WORKERS=4 npx vitest related src/a.ts`: 12 passed), and each acceptance criterion with how it was checked.

## Review findings
(A fix round only.) Each finding, by file:line, and what was done about it.

## Left
Readings taken where the ticket was silent, anything not done, and known issues. "Nothing" when nothing is.
```

## 7. Report

Call `report_outcome` on the `cockpit` MCP server (`mcp__cockpit__report_outcome`) with a one-line summary ("Ticket 03 built: failed steps pause the run; 6 tests added").
