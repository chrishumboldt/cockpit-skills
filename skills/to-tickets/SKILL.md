---
name: to-tickets
description: The Break into tickets step of a Cockpit run. Breaks spec.md into tracer-bullet tickets, one file each in the run folder, with a human index (tickets.md) and a machine index (tickets.json); reports READY.
disable-model-invocation: false
---

# To tickets

You run the Break into tickets step of a Cockpit run. Break the approved spec into **tickets**: tracer-bullet vertical slices, each naming the tickets that **block** it. You ask nothing; the approval gate after you is where the user weighs the breakdown. Every file you write goes in the run folder the opening message names.

## 1. Read

`spec.md` in the run folder (the prompt gives its path), then the code it touches, the glossary and the ADRs in that area. Look for prefactoring that makes the change easy: "make the change easy, then make the easy change".

## 2. Slice

- Each ticket cuts a narrow but **complete** path through every layer it needs (data, logic, interface, tests): vertical, never one layer of many features.
- A finished ticket is demoable or checkable on its own.
- Each fits one fresh context window: an agent can build and test it in one sitting.
- Prefactoring comes first, as its own ticket.
- A ticket is **blocked by** only the tickets that genuinely gate it. A ticket with no blockers can start at once, and tickets with no edge between them may be built at the same time, so keep false edges out.
- Every behaviour and acceptance criterion in the spec lands in some ticket.

**Wide refactors are the exception.** One mechanical change whose blast radius spans the codebase (renaming a shared type, a column) can't land green as one vertical slice. Sequence it **expand–contract**: first a ticket that adds the new form beside the old, so nothing breaks; then migrate the callers in batches sized by blast radius (per package, per folder), each its own ticket blocked by the expand, each green on its own because the old form still exists; last, a contract ticket that deletes the old form, blocked by every batch.

## 3. Write the files

Number the tickets from `01` in dependency order, blockers first, two digits. The slug is the title in lowercase kebab case.

**One file per ticket**, `tickets/NN-<slug>.md`:

```markdown
# NN: <Title>

**Blocked by:** NN, NN (or "None")

## What to build
The end-to-end behaviour this ticket makes work, as the person using it sees it; not a list of layers.

## Acceptance criteria
- [ ] <criterion, true or false by looking>
- [ ] …

## Verified by
The tests that prove it, by seam and behaviour ("an engine test: a failed step pauses the run"), and any check by hand or end-to-end spec.
```

**The human index**, `tickets.md`, the file the approval gate shows:

```markdown
# Tickets · <spec title>

<N> tickets from `spec.md`, in build order.

## 01 · <Title>
**Blocked by:** None
**Delivers:** one or two lines on what works once it's done.
**File:** `tickets/01-<slug>.md`

## 02 · …
```

**The machine index**, `tickets.json`, the ticket set the engine reads:

```json
[
  { "id": "01", "slug": "<slug>", "title": "<Title>", "blockedBy": [], "file": "tickets/01-<slug>.md" },
  { "id": "02", "slug": "<slug>", "title": "<Title>", "blockedBy": ["01"], "file": "tickets/02-<slug>.md" }
]
```

The three agree exactly: the same tickets, numbers, titles and edges, `file` relative to the run folder, `blockedBy` only ever naming earlier ids. Name modules and seams by their role; give a file path or a snippet only where it pins a decision.

## 4. When the gate sends it back

When the user requests changes, the message lists their notes (also kept in `gate-tickets.md`). Rework the breakdown to address every note and rewrite all three: renumber when the order changes, and delete any `tickets/*.md` file whose ticket is gone, so the folder holds exactly the tickets in `tickets.json`.

## 5. Report

Call `submit_artifact` on the `cockpit` MCP server with kind `tickets` and the path of `tickets.md`, then `report_outcome` (`mcp__cockpit__report_outcome`) with outcome `READY` and a one-line summary ("7 tickets, 3 can start at once").
