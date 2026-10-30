---
name: diagnose
description: The Diagnose step of Bug 2 Shipped. Reproduces a bug with a failing test, finds its root cause, commits the test and writes diagnosis.md for the fix; reports FOUND, or NOT_FOUND when it can't reproduce it.
disable-model-invocation: false
---

# Diagnose

You open a Bug 2 Shipped run. Prove the bug, find why it happens and hand both to the Implement step: a failing test committed on the run's branch, and `diagnosis.md`. You fix nothing; a fix written before the cause is known patches the symptom.

## 1. Read
- The bug as the user told it: the step's opening message, and `transcript-diagnose.md` in the run folder when Cockpit passed the home chat.
- The repo's agent docs (`CLAUDE.md`, `AGENTS.md`) for its test commands, and the code the bug points at.

Ask the user only what the repo can't tell you, one short question at a time: the steps that show it, what they expected, what happened. Stop asking once you can try to reproduce it.

## 2. Reproduce it with a test
Write the smallest test that fails because of the bug, at the lowest seam that shows it (a unit test over an end-to-end one when both can). Run it and watch it fail for the reason the user saw, not for a typo or a missing import. When it passes, you haven't reproduced the bug yet: change the test, not your story.

## 3. Find the root cause
Follow the failure back to the line where the behaviour first goes wrong: read the code, add a log or an assertion, bisect the history (`git log -S`, `git bisect`) when it used to work. The cause is the first wrong decision in the code, not where the error surfaces. Check the cause explains everything the user saw; a cause that explains only part of it isn't the cause yet. Look for the same mistake elsewhere and list where it is.

Take out any logging you added. Commit the failing test alone on the current branch: "Failing test: <the bug, in a few words>". The Implement step makes it pass.

## 4. Write `diagnosis.md`
The path the prompt gives:

```markdown
# Diagnosis · <the bug, in a few words>

## Bug
What the user does, what they expect, what happens.

## Reproduced by
`path/to/test:line`, the command that runs it, and its failure output (the few lines that matter).

## Cause
`file:line`: what goes wrong there and why, and how it leads to what the user saw.

## Fix
The smallest change that removes the cause, and where. Other places with the same mistake, when the fix should cover them.

## Not the cause
What looked likely and was ruled out, and how.
```

## 5. When you can't
When you can't reproduce the bug or can't find its cause, commit nothing, write `diagnosis.md` with what you tried and what would help (a log, an environment, steps), and report `NOT_FOUND`. A guessed cause is worse than none.

## 6. Report
Call `report_outcome` on the `cockpit` MCP server (`mcp__cockpit__report_outcome`) with `FOUND` or `NOT_FOUND` and a one-line summary ("Reproduced: the queue drops a message sent mid-turn; cause in message-queue.ts:43").
