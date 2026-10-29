---
name: backlog-add
description: Add an item to the project's backlog (docs/backlog/) from a few words in the chat, without a workflow: for small, clear changes such as a UX fix or a shortcut. Writes the item, commits it to the base branch through a temporary worktree and pushes it. Use when asked to "add to the backlog", "note that for later" or "file that".
disable-model-invocation: false
---

# Backlog add

You add one or more items to the backlog straight from the conversation. Use this for small, clear things. When an idea needs questions answered, a design or a spec, say so and suggest the `Idea 2 Backlog` workflow instead.

## 1. Read
`docs/backlog/README.md` (the format; when the backlog doesn't exist yet, the format is `format.md` beside the `backlog-items` skill, and you copy it to `docs/backlog/README.md`), the existing items (their numbers, and any this overlaps or depends on), and enough code to name where the change goes.

## 2. Ask only what you must
If the person's words leave the behaviour unclear, ask one short question. Otherwise write it. Set `status: ready` when it's clear enough to build, `idea` when it isn't, and say why in the item.

## 3. Write and land, without touching the person's checkout
The checkout the session works in may be a running app or have work in progress, so never write there:

```sh
base=$(git symbolic-ref --short refs/remotes/origin/HEAD | sed 's#^origin/##')
git fetch -q origin "$base"
tmp=$(mktemp -d) && git worktree add -q --detach "$tmp" "origin/$base"
# write docs/backlog/NNN-<slug>.md in "$tmp" (and README.md when new)
git -C "$tmp" add docs/backlog && git -C "$tmp" commit -q -m "Backlog: NNN <title>"
git -C "$tmp" push -q origin "HEAD:$base"
git worktree remove "$tmp"
```

If the push is refused because the base moved, fetch, rebase the one commit and push again. Number from the highest item on the fetched base.

## 4. Say what you added
One line per item: its number, title, status, and `blockedBy` / `needs` when set.
