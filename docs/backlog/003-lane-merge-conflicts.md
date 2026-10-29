---
status: done
kind: bug
area: workflows
added: 2026-10-29
done: 2026-10-30
---

# 003 · A lane merge conflict stops the run, and Retry can't clear it

In Backlog 2 Shipped (and Idea 2 Shipped, which has the same lane steps) each lane ends with `merge: git merge --no-ff {{item.branch}}` into the run branch. Lanes build side by side, so two items that change the same file conflict when the second one merges. In run 8c304e on 2026-10-29 it happened twice: lanes 03 and 04 (the package store and `update.ts`, plus two new ADRs both numbered 0040), then lanes 08 and 07 (`SkillsView.svelte`; four of the picked items changed the Skills view). Each time:
- The failed merge leaves the run worktree mid-merge. `on_stop: git merge --abort` runs only on Stop lane, so **Retry** fails at once with "Merging is not possible because you have unmerged files" (exit 128) and never retries.
- Nothing in the workflow resolves a conflict: the user has to take **Fix it myself**, resolve it by hand (or with another agent), commit and press Done.
- Pick items didn't see the overlap coming: items that change the same files ran in parallel instead of one after another.

## What to build
- **A failed lane merge cleans up after itself.** The merge step aborts a conflicted merge before it fails (`git merge --no-ff … || { git merge --abort; exit 1; }`), so the run worktree is clean and Retry really retries. The log still names the conflicted files.
- **A conflicted merge goes to an agent before it goes to the user.** On a conflict the lane runs an agent step (role implementer, skill `cockpit/implement` or a small new one) in the run worktree: merge again, resolve keeping both sides against the two items' tickets and the spec, renumber clashing ADRs (or any other numbered files) to the next free number with their references, run the focused tests, commit the merge. If it can't, the lane fails to the user as now.
- **Pick items orders overlapping items.** `backlog-pick` (and `backlog-items` when it writes the run's tickets) looks at the files and areas each ticket will change. When two tickets will clearly touch the same files, it makes the later one `blockedBy` the earlier, so they build one after another, and says so in its report line.

## Acceptance criteria
- [ ] A lane whose merge conflicts leaves the run worktree clean (no unmerged files, no MERGE_HEAD), and Retry runs the merge again.
- [ ] A conflict that keeps both sides is resolved and committed by the agent step without a gate; one it can't resolve still reaches the user with Fix it myself.
- [ ] Two lanes that each add an ADR numbered the same end up with different numbers and matching references.
- [ ] Picking two items that change the same file makes one wait for the other in `tickets.json`.
- [ ] Both workflow files and the skills' frontmatter still parse.

## Verified by
A Backlog 2 Shipped run in a scratch project picking two items that edit the same function: the second lane's merge conflicts, the agent resolves and commits it, and the run reaches Final review without a gate. Then the same run with the agent step failing: the run worktree is clean and Retry merges again.
