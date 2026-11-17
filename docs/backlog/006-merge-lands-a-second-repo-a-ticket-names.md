---
status: ready
kind: feature
area: workflows
added: 2026-11-17
---

# 006 · Merge lands a second repo a ticket names

## What to build
A ticket sometimes changes a repo other than the project's, on a branch
of its own, and that branch has to land with the project's change. On
2026-11-17 Cockpit's run 1557fd (one command grammar) built ticket 09 on
branch `grammar-1557fd` of cockpit-skills and wrote "the Merge step
pushes cockpit-skills first"; but Merge (`land` in every workflow) only
merges and pushes the project's branch, so the skills branch was left
local until the coordinator pushed and merged it by hand. Meanwhile the
app refused the old `skill:` form, so the installed skillset would have
stopped parsing.

- **A lane declares a second repo in its hand-off.** When `implement`
  builds on another repo, it clones it in the lane folder and writes
  `handoff.json` beside `build.md`: the clone's path, the remote, the
  branch it made and the base branch to land on. The `implement` skill
  documents the file and when to write it.
- **Merge lands it first.** Before merging the project's branch, the
  `land` step of Backlog 2 Shipped, Idea 2 Shipped and Bug 2 Shipped
  reads every lane's `handoff.json`, pushes the branch, merges it into
  the base with one merge commit ("Merge <branch> into <base>", noting
  the run) and pushes the base, then goes on to the project. A failure
  there stops the step before anything lands in the project, with the
  repo and the error in the log.
- **The contract says so.** `docs/contract.md` names `handoff.json`; the
  `write-workflow` skill's format notes it so a project's own workflow
  can keep the behaviour.
- The installed pin in `~/.cockpit/packages.json` is not moved: the
  user takes the update in the app.

## Acceptance criteria
- [ ] A lane that writes `handoff.json` has its second-repo branch
  pushed and merged into that repo's base, with a merge commit, before
  the project's branch merges.
- [ ] A lane with no `handoff.json` lands as today.
- [ ] A failed second-repo push or merge fails the Merge step with the
  repo named, and the project's branch is not merged.
- [ ] The implement skill, the contract and the write-workflow format
  describe the file.

## Verified by
The workflows' land scripts run against two throwaway git repos (a
project and a second repo with a lane branch): the second repo's main
gains the merge commit first; the project's base after it; the no-file
case unchanged; the failing case leaves the project base untouched. A
check by hand: a Backlog 2 Shipped run whose ticket changes a second
repo.
