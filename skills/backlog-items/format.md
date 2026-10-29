# The backlog format

The backlog is everything waiting to be built in a project, one Markdown file per item, in `docs/backlog/`. It replaces an issue tracker: `Idea 2 Backlog` and `backlog-add` write items, and `Backlog 2 Shipped` builds them and marks them done.

## Files

- `docs/backlog/README.md`: this format, copied in when the backlog is first made.
- `docs/backlog/NNN-<slug>.md`: one item. `NNN` is three digits, never reused; a new item takes the highest number plus one. The slug is the title in lowercase kebab case.
- `docs/backlog/specs/<slug>.md`: a spec from `Idea 2 Backlog`, shared by the items it split into.

## An item

```markdown
---
status: ready
kind: feature
area: settings
blockedBy: [012]
needs: The project's Setup command is set
spec: specs/open-links-with.md
shots: settings
added: 2026-10-29
---

# 014 · Open links with a chosen browser

## What to build
The behaviour as the person using it sees it, end to end.

## Acceptance criteria
- [ ] True or false by looking.

## Verified by
The tests that prove it, and any check by hand.
```

| Field | Required | Meaning |
|---|---|---|
| `status` | yes | `idea` (written down, not ready: it needs shaping or a decision), `ready` (clear enough to build) or `done` (built and on the base branch; the file stays, as history). |
| `kind` | yes | `feature`, `ux` or `bug`. |
| `area` | no | One word that groups items (`thread`, `settings`). |
| `blockedBy` | no | Item numbers that must be `done` first. |
| `needs` | no | Free text for what isn't an item: a setting, a package update, a decision. Shown when picking, never enforced. |
| `spec` | no | The spec the item came from, relative to `docs/backlog/`. |
| `shots` | no | The project's screenshot names for the screens it touches, comma-separated; a workflow of your own may screenshot them. |
| `added` | yes | The date it was written. |
| `done` | no | The date it landed, set by `Backlog 2 Shipped`. |

The body has the same three sections as a ticket, so the `implement` skill builds an item as it builds a ticket.
