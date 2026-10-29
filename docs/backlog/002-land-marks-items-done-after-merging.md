---
status: ready
kind: bug
area: workflows
added: 2026-10-29
---

# 002 · Merge marks items done after merging, on the fetched base

Backlog 2 Shipped's Merge step (`land` in `workflows/backlog-2-shipped.yaml`) marks the picked items done on the run branch *before* it fetches the base and merges. A run branch starts from the base as the project's checkout had it, while `backlog-add` pushes new items straight to the remote base. So an item filed after the checkout last pulled is on the remote but not on the run branch: Pick items offers it and the lanes build it, then Merge's `sed` can't find the file, `set -e` stops the script (exit 2) and nothing lands. It happened twice on 2026-10-29; the way round it was merging the remote base into the run branch by hand and retrying. An item edited on the base during a run has the same problem in a milder form: the run branch's copy is stale.

## What to build
- In the `land` step, mark the items done *after* `git checkout --detach origin/$base` and the merge, on the merged tree, where every item on the base exists: edit `status`/`done`, commit "Backlog: mark … done" on top of the merge, then push. Inside the retry loop, so a retry after the base moved marks them again on the new base.
- An item in `picks.txt` that isn't on the merged tree either is skipped with a line in the log ("033 not found; not marked"), not a failure: the code still lands.
- The same change in `workflows/idea-2-shipped.yaml`'s `land` step, which has the same script (there the items are written on the run branch, so it works today, but the two should stay one script).
- `git checkout -q "$branch"` at the end still leaves the worktree on the run branch.

## Acceptance criteria
- [ ] A run that picks an item filed on the remote base after the run started lands: the merge succeeds and the item reads `status: done` with today's `done:` on the base.
- [ ] An item edited on the base during the run keeps that edit and is marked done.
- [ ] A missing item is logged and skipped; the merge still lands.
- [ ] When the push is refused because the base moved, the retry marks the items on the new base and the base ends with one merge commit and one marking commit.
- [ ] Both workflow files parse as YAML and their `land` scripts match.

## Verified by
A Backlog 2 Shipped run in a scratch project whose checkout is one commit behind its remote, picking an item that exists only on the remote: Merge lands and the item is done on the base. The YAML parsed after the edit.
