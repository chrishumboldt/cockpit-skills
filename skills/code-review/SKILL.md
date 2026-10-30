---
name: code-review
description: A review step of a Cockpit run. Reviews a change in one of two modes the step prompt picks, correctness against the spec and ticket or code smells only, writes the review file the prompt names and reports PASS or CHANGES.
disable-model-invocation: false
---

# Code review

You run a review step of a Cockpit run: read one change and judge it on one axis. You fix nothing and ask nothing; the review file is your whole output, and an Implement step acts on it.

## 1. Pick the mode

The step prompt says which:

- **Correctness** (the default): does the change do what the spec and the ticket ask, and does it work?
- **Smells only**, when the prompt says code smells: is the change well made? Correctness is another reviewer's job here; leave it alone.

## 2. Find the change

- A ticket's change is the work on the current branch: its commits since it left the branch it was made from (`git merge-base HEAD <that branch>`; the prompt names it) plus anything uncommitted.
- A branch "as a whole" (the final review) is the diff from its merge-base with the base the prompt names to its tip.
- Check the range resolves and isn't empty before reviewing. An empty change is a finding of its own: report `CHANGES` with "Nothing to review on <range>".

Note the range and the commit list; the review names them.

## 3. Review

**Correctness.** Read the ticket file (its What to build and acceptance criteria) and `spec.md` (the behaviours and decisions it touches), then the diff and enough of the code around it to know it works. Look for:

- an acceptance criterion or spec behaviour missing or only partly done;
- behaviour that is wrong: a bug, an edge case or error path mishandled, a race, a broken caller;
- behaviour nobody asked for (scope creep);
- tests that don't prove what Verified by says, or that pass by construction;
- a break with the repo's documented standards (`CONTRIBUTING.md`, `CODING_STANDARDS.md`, the agent docs, the ADRs) the tooling doesn't already enforce.

**Smells only.** Read the diff against the list in [smells.md](smells.md) and the repo's own standards; a documented repo standard wins over the list. Every smell is a judgement call: name it as "possible Feature Envy", never as a rule broken.

When the step reviews a fix round, an earlier review's findings come first: check each one was addressed.

## 4. Write the review

Write the file the prompt names. Its first line is the outcome, exactly `OUTCOME: PASS` or `OUTCOME: CHANGES`, with nothing before it:

```markdown
OUTCOME: CHANGES

# Code review · <ticket NN: Title, or the branch>
Mode: correctness (or smells). Range: `<base>..<head>`, <n> commits. Against: `<ticket file>`, `<spec.md>`.

## Changes
1. `src/runs/engine.ts:142`: <the problem, concretely>. → <what to change>.
2. …

## Optional
1. `src/runs/texts.ts:30`: <a smaller point>. → <the suggestion>.

## Checked
- <what was looked at and holds up>, one line each, so the reader knows what passed.
```

Every finding gives the file and line, the problem and what to change. **Changes** holds what must change before this ships; **Optional** what would be better but needn't block. Leave a section out when it is empty.

The outcome is `CHANGES` when Changes has any finding, else `PASS`.

## 5. Report

Call `submit_artifact` on the `cockpit` MCP server with kind `review` and the review's path, then `report_outcome` (`mcp__cockpit__report_outcome`) with `PASS` or `CHANGES`, matching the file's first line, and a one-line summary ("2 changes: a missed edge case and an untested error path").
