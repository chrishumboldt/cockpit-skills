---
name: resolve-merge
description: The Resolve merge step of a lane in Backlog 2 Shipped and Idea 2 Shipped. Merges a lane's branch into the run branch again after the plain merge conflicted, resolves the conflicts keeping both sides, renumbers clashing numbered files, runs the focused tests and commits the merge; reports DONE.
disable-model-invocation: false
---

# Resolve merge

You run the Resolve merge step of a lane. The lane's plain merge into the run branch hit a conflict and aborted, so two items changed the same thing. Merge the run branch into the lane's branch and resolve it so both items' work survives. You ask nothing: where the tickets and spec are silent, take the reading that fits them best and say so in `resolve.md`.

## 1. Work in the lane's worktree
Stay in your working directory, the lane's own worktree, on the lane branch. Never touch the run worktree: other lanes merge there and it can't be held for you. The lane worktree should be clean; if it is mid-merge, an earlier try of this step left it, perhaps on an older run-branch tip, so run `git merge --abort` and start again from step 2.

## 2. Merge the run branch into the lane
`git merge --no-ff --no-commit <run branch>`. List the conflicted files with `git diff --name-only --diff-filter=U`. The merge may go through with no conflicts when the only clash is numbered files (two `0040-` ADRs with different slugs); then step 3's renumbering goes into the same merge commit. The Merge step that follows then merges the lane into the run branch cleanly.

## 3. Resolve, keeping both sides
- Read the lane's ticket and the other items' tickets that already merged (`tickets/*.md` in the run folder) and `spec.md`, so you know what each side meant. The run branch's side is the other items'.
- Resolve each conflict so the result does what both items ask. Never drop a side to make the conflict go away; if the two really can't both hold, stop at step 5.
- **Numbered files** (ADRs, migrations, anything named `NNNN-…`; dates like `2026-10-30-…` are not sequence numbers, so when the only clash is dated files, merge without renaming): when both sides added a file with the same number, keep the one already on the run branch and renumber the lane's to the next free number in that folder. Update the number in its title, its filename and every reference to it (other docs, code comments, the glossary), and search the whole tree for the old number to find them.

## 4. Test and commit
Run the type checker and the focused tests for the files the merge touched, with the repo's own commands and worker caps, as the Implement skill's Verify step finds them. Fix what the merge broke. Then `git add` the resolved files and `git commit --no-edit` to finish the merge on the lane branch. Write `resolve.md` in the lane folder: the files that conflicted, how each was resolved, any renumbering, the tests run.

## 5. When you can't
If the two sides can't both be kept, or the tests won't pass after the merge, run `git merge --abort`, leave the worktree clean and write `resolve.md` with why. Call `report_outcome` with `DONE` and a summary that starts "Couldn't resolve: …". The Merge step that follows conflicts again and fails to the user, who can take Fix it myself.

## 6. Report
Call `report_outcome` with `DONE` and a one-line summary ("Resolved 2 conflicts in SkillsView.svelte; ADR 0040 renumbered to 0041").
