---
status: done
kind: feature
area: backlog-pick
added: 2026-10-29
done: 2026-10-29
---

# 001 · Pick items flags stale items before writing tickets

An item is written against the code as it was that day. Later work can do it, supersede it, or move what it names, and nothing notices: the Pick items step checks blockers and `needs`, never whether the item still makes sense against the base branch. A run then builds a change that's already there, or edits code that has moved, and the waste shows up only in the lane. Example: two items on the same day, one renaming a tab and the next renaming it back; a run picking only the first would have undone the second.

## What to build
In `skills/backlog-pick/SKILL.md`, step 2 (Check the pick) gains a staleness check for each picked item, before any ticket is written:
- Read the files, symbols and labels the item names, as they are on the base branch now.
- Look at what has landed since the item's `added` date: later backlog items (including `done` ones) that touch the same thing, and the base branch's commits that touched those files (`git log --since=<added> -- <files>`).
- Flag the item, with one line of evidence each, when:
  - **Done already**: the base already behaves as the acceptance criteria say.
  - **Superseded**: a later item reverses or replaces it.
  - **Moved**: a file, symbol or label it names no longer exists or has been renamed.
- For a flagged item, ask the user: build it anyway, drop it from this run, or mark it so the backlog says so (set `status: idea` with a line on why, or `done` with the date when it's already there; that edit goes in the run's commit with the tickets). An item that isn't flagged goes through without a question.
- The check reads; it never runs builds or tests.

The step's report line names any flagged items and what the user chose ("3 items: 014, 015, 016; 017 dropped, already done").

## Acceptance criteria
- [ ] Picking an item whose change is already on the base flags it as done already, with the evidence, and asks before writing its ticket.
- [ ] Picking an item that a later item reverses flags it as superseded and names the later item.
- [ ] Picking an item whose named file or label no longer exists flags it as moved.
- [ ] Items with no flag are written as tickets without an extra question.
- [ ] The report line names flagged items and the choice made.
- [ ] The skill's frontmatter still parses (no unquoted `: ` in a value).

## Verified by
A Backlog 2 Shipped run in a project with one item already done on the base, one superseded by a later item and one clean: the first two are flagged with their evidence, the clean one isn't; the report line lists the choices.
