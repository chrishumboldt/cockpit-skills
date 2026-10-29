---
name: backlog-pick
description: The Pick items step of Backlog 2 Shipped. Lists the ready backlog items, asks the user which to build, checks blockers, needs and staleness, and writes the run's tickets, spec.md and picks.txt; reports READY, or NONE when nothing is picked.
disable-model-invocation: false
---

# Backlog pick

You run the Pick items step of a Cockpit `Backlog 2 Shipped` run. You are interactive: the user answers you in the chat. The format is in `docs/backlog/README.md`.

## 1. Offer
Read every item in `docs/backlog/`. Use the session's question card (your `AskUserQuestion` tool) when the answer is a pick between options, and plain text when it needs the user's words.

- **No `ready` item:** write no tickets and report `NONE` without asking.
- **1–4 `ready` items:** ask with one multi-select question card, one option per ready item. A card needs at least 2 options, so with a single ready item the options are that item ("NNN · Title") and "None". The option's label is the number and title; its description gives kind, area, and `blockedBy` / `needs` when set, marking one whose blockers aren't done ("needs 014 first"). Put one line on how many `idea` items wait for shaping in the question text. The picked options are the pick; ticking "None" means nothing is picked. An answer typed in the card's Other field ("all ux", "1-3") is handled like a typed answer below; typed numbers are backlog item numbers, not card positions.
- **More than 4 `ready` items** (a card holds at most 4 options): list them as a numbered list, one line each with the same details, and under it one line on how many `idea` items wait for shaping. Ask in plain text which to build; the answer may be numbers, a range or a filter ("all ux").

## 2. Check the pick
Ask each of these with a yes / no question card.

- An item whose blockers aren't done and aren't picked: name them and ask whether to add them (text typed in Other is taken as the user's words).
- An item with `needs`: show it and ask whether it's met.
- An item too thin to build safely (no clear behaviour or acceptance, or plainly bigger than one sitting): say so and suggest shaping it with `Idea 2 Backlog` first, and ask whether to build it anyway.
- Staleness, for each picked item, before any ticket is written. Read the files, symbols and labels it names as they are on `origin/<base>` after a `git fetch`, and what has landed since its `added` date: later backlog items (`done` ones too) that touch the same thing, and the base's commits on those files (`git log --since=<added> -- <files>`). Flag the item, with one line of evidence, when it is:
  - **Done already**: the base already behaves as the acceptance criteria say.
  - **Superseded**: a later item reverses or replaces it; name that item.
  - **Moved**: a file, symbol or label it names no longer exists or was renamed.

  Ask about each flagged item with a question card: build it anyway, drop it from this run, or mark it in the backlog (`status: idea` with a line on why, or `done` with the date when it's already there). Edit a marked item and commit it on the run's branch (`git add docs/backlog/<file> && git commit -m "Backlog: mark NNN <status>"`), so the land step merges it in. If nothing is left to build, commit the mark to the base the way `backlog-add` does (temporary worktree on the fetched base, push), then report `NONE`. A dropped or marked item is left out of the tickets, `tickets.json`, `spec.md` and `picks.txt`; only items being built go in them. An item with no flag goes through without a question. This check only reads; it never runs builds or tests.
- Nothing picked, or nothing ready: write no tickets and report `NONE`.

## 3. Write the run's tickets
In the run folder, in build order (blockers first):
- `tickets/NN-<slug>.md` per item: `# NN: <NNN · title>`, `**Blocked by:**` mapped to the run's ticket ids, then the item's What to build, Acceptance criteria and Verified by.
- `tickets.json`, the ticket set the engine reads: a bare JSON array, never an object around it, one entry per ticket with its item number:
  ```json
  [
    { "id": "01", "item": "014", "slug": "<slug>", "title": "<Title>", "blockedBy": [], "file": "tickets/01-<slug>.md" },
    { "id": "02", "item": "015", "slug": "<slug>", "title": "<Title>", "blockedBy": ["01"], "file": "tickets/02-<slug>.md" }
  ]
  ```
  `blockedBy` names only earlier ticket ids; `file` is relative to the run folder.
- **Order items that overlap.** Lanes build side by side and their merges conflict when two tickets change the same file. For each ticket, note the files and areas it will change (from its What to build and the code it names). When two tickets will clearly touch the same files, make the later one `blockedBy` the earlier (reorder if needed so the blocker comes first), so they build one after another, and put the edge in both `tickets.json` and the ticket's `**Blocked by:**`. Don't add an edge for a loose area match; a real conflict goes to the Resolve merge step. Say each added edge in the report line.
- `tickets.md`, the human index, as the `to-tickets` skill writes it, each ticket naming its item number.
- `spec.md`, which the build steps read: titled "Build · <item numbers>", then the specs the items name (`docs/backlog/specs/…`), each once, and for an item with no spec its own body.
- `picks.txt`: the picked items' paths, one per line, relative to the repository root. The land step marks them done.

## 4. Report
Call `report_outcome` with `READY` and a one-line summary ("3 items: 014, 015, 016; 016 waits on 014"); say which waits are for overlapping files ("016 waits on 015: both change SkillsView.svelte"). Name any flagged items and what the user chose ("3 items: 014, 015, 016; 017 dropped, already done").
