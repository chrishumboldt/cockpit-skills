---
name: resolve-merge
description: The Resolve merge step of a lane in Backlog 2 Shipped and Idea 2 Shipped. Merges a lane's branch into the run branch again after the plain merge conflicted, resolves the conflicts keeping both sides, renumbers clashing numbered files, runs the focused tests and commits the merge; reports DONE.
disable-model-invocation: false
---

# Resolve merge

You run the Resolve merge step of a lane. The lane's plain merge hit a conflict and aborted, so two items changed the same thing. Merge the lane's branch into the run branch again and resolve it so both items' work survives. You ask nothing: where the tickets and spec are silent, take the reading that fits them best and say so in `resolve.md`.

## 1. Find the run worktree
The merge belongs on the run branch (the prompt names it). If your working directory has another branch checked out, find the worktree that has the run branch with `git worktree list` and work there. It should be clean and not mid-merge; if it is mid-merge, a person or an earlier try left it, so carry on from that merge rather than starting over.

## 2. Merge again
`git merge --no-ff --no-edit <lane branch>`. List the conflicted files with `git diff --name-only --diff-filter=U`.

## 3. Resolve, keeping both sides
- Read the lane's ticket and the other items' tickets that already merged (`tickets/*.md` in the run folder) and `spec.md`, so you know what each side meant.
- Resolve each conflict so the result does what both items ask. Never drop a side to make the conflict go away; if the two really can't both hold, stop at step 5.
- **Numbered files** (ADRs, migrations, anything named `NNNN-…`): when both sides added a file with the same number, keep the one already on the run branch and renumber the lane's to the next free number in that folder. Update the number in its title, its filename and every reference to it (other docs, code comments, the glossary), and search the whole tree for the old number to find them.

## 4. Test and commit
Run the type checker and the focused tests for the files the merge touched (`vitest related <files>` and the like), with the workers capped (`VITEST_MAX_WORKERS=4`). Fix what the merge broke. Then `git add` the resolved files and `git commit --no-edit` to finish the merge. Write `resolve.md` in the lane folder: the files that conflicted, how each was resolved, any renumbering, the tests run.

## 5. When you can't
If the two sides can't both be kept, or the tests won't pass after the merge, run `git merge --abort`, leave the worktree clean and write `resolve.md` with why. Call `report_outcome` with a summary that says it failed, so the lane fails to the user.

## 6. Report
Call `report_outcome` with `DONE` and a one-line summary ("Resolved 2 conflicts in SkillsView.svelte; ADR 0040 renumbered to 0041").
