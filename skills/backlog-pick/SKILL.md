---
name: backlog-pick
description: The Pick items step of Backlog 2 Shipped. Lists the ready backlog items, asks the user which to build, checks blockers and needs, and writes the run's tickets, spec.md and picks.txt; reports READY, or NONE when nothing is picked.
disable-model-invocation: false
---

# Backlog pick

You run the Pick items step of a Cockpit `Backlog 2 Shipped` run. You are interactive: the user answers you in the chat. The format is in `docs/backlog/README.md`.

## 1. Offer
Read every item in `docs/backlog/`. List the `ready` ones as a numbered list, one line each: number, title, kind, area, and `blockedBy` / `needs` when set; mark one whose blockers aren't done ("needs 014 first"). Under it, one line on how many `idea` items wait for shaping. Ask which to build; the answer may be numbers, a range or a filter ("all ux").

## 2. Check the pick
- An item whose blockers aren't done and aren't picked: name them and ask whether to add them.
- An item with `needs`: show it and ask whether it's met.
- An item too thin to build safely (no clear behaviour or acceptance, or plainly bigger than one sitting): say so and suggest shaping it with `Idea 2 Backlog` first, unless the user wants it built anyway.
- Nothing picked, or nothing ready: write no tickets and report `NONE`.

## 3. Write the run's tickets
In the run folder, in build order (blockers first):
- `tickets/NN-<slug>.md` per item: `# NN: <NNN · title>`, `**Blocked by:**` mapped to the run's ticket ids, then the item's What to build, Acceptance criteria and Verified by.
- `tickets.json` and `tickets.md`, as the `to-tickets` skill writes them, each ticket naming its item number.
- `spec.md`, which the build steps read: titled "Build · <item numbers>", then the specs the items name (`docs/backlog/specs/…`), each once, and for an item with no spec its own body.
- `picks.txt`: the picked items' paths, one per line, relative to the repository root. The land step marks them done.

## 4. Report
Call `report_outcome` with `READY` and a one-line summary ("3 items: 014, 015, 016; 016 waits on 014").
