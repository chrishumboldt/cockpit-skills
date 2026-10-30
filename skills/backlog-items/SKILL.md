---
name: backlog-items
description: The Write backlog items step of Idea 2 Backlog and Idea 2 Shipped. Splits an approved spec into backlog items in docs/backlog/ with their dependencies, saves the spec beside them, commits on the run's branch and writes items.md for the gate; reports READY.
disable-model-invocation: false
---

# Backlog items

You run the Write backlog items step of a Cockpit `Idea 2 Backlog` run. You ask nothing; the Approve items gate after you is where the user weighs the split. The format is in `format.md` beside this skill; when the project has no `docs/backlog/README.md` yet, copy `format.md` there.

## 1. Read
`spec.md` in the run folder, the existing items in `docs/backlog/` (their numbers, and any this spec overlaps, depends on or replaces), then the code the spec touches.

## 2. Split
Slice the spec as the `to-tickets` skill does: vertical slices, each complete and checkable on its own, each one sitting of work, prefactoring first, `blockedBy` only where an item truly gates another. Every behaviour and acceptance criterion lands in some item. A small spec is one item.

- `blockedBy` names item numbers: the new items' own, and existing items this needs done first.
- `needs` is free text for what isn't an item.
- `status` is `ready`, unless an open question in the spec decides the item: then `idea`, naming the question.

## 3. Write
- Number from the highest item plus one, three digits, counting both the run's branch and the base as it is now, since other sessions add items while a run goes: `git fetch -q origin && git ls-tree --name-only "origin/<base>" docs/backlog/`, with the base the prompt names.
- Copy `spec.md` to `docs/backlog/specs/<spec slug>.md`; each item's `spec:` names it.
- End that copy with a `## Backlog items` section listing the items it became, one line each: `- 014 · <Title>` (`NNN-<slug>.md`). Someone who opens the spec finds its items there; whether they're done stays in each item's `status`.
- Each item: `docs/backlog/NNN-<slug>.md` in the format, `added:` today.
- Commit on the run's branch: `git add docs/backlog && git commit -m "Backlog: <spec title>"`.
- Write `items.md` in the run folder, the file the gate shows:

```markdown
# Backlog items · <spec title>

<N> items from `spec.md`, the spec saved as `docs/backlog/specs/<slug>.md`.

## 014 · <Title>
**Status:** ready · **Kind:** feature · **Blocked by:** none · **Needs:** none
**Delivers:** one or two lines.
```

## When the prompt asks for the run's tickets too
In `Idea 2 Shipped` the run builds the items you wrote right after the gate. Then also write, in the run folder, what its fan-out reads, exactly as the `backlog-pick` skill describes in its step 3: `tickets/NN-<slug>.md` per item, `tickets.json` (a bare JSON array of `{ id, item, slug, title, blockedBy, file }`, never an object around it), `tickets.md`, and `picks.txt` (the items' repo paths). Order overlapping tickets as `backlog-pick` does: when two tickets will clearly touch the same files, the later one is `blockedBy` the earlier. Keep `spec.md` as it is. On a revision from the gate, rewrite them along with the items.

## 4. Back from the gate
Request changes resumes you with the notes in `gate-items.md`: revise the items and `items.md` in place, commit again, and report again.

## 5. Report
Call `report_outcome` with `READY` and a one-line summary ("3 items, 014 to 016; 015 and 016 need 014").
