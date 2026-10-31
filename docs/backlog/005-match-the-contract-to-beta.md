---
status: idea
kind: bug
area: workflows
added: 2026-10-31
---

# 005 · Match the contract to beta

This turns `ready` only once beta (branch `beta` of chrishumboldt/cockpit) is merged into cockpit's `main`: it describes beta's engine, and alpha still runs these files until then.

The contract and the workflow-writing format describe alpha's engine. Beta accepts a narrower set of variables, names lane branches differently, decides which steps resume their session, and the land steps don't clean up a failed merge when stopped.

## What to build
- **The contract's variables.** `docs/contract.md:13` lists only the nine variables beta accepts: `{{run.folder}}`, `{{run.branch}}`, `{{run.base}}`, `{{<step>.file}}`, `{{item.id}}`, `{{item.title}}`, `{{item.file}}`, `{{item.branch}}` and `{{lane.folder}}`. It drops `{{item.slug}}`, `{{<step>.title}}` and `{{<step>.preface}}`, and states that any other `{{…}}` is a parse error naming the file, the step and the variable.
- **The lane branch.** The contract names the lane branch `cockpit/<run-id>-<item.id>-<item.slug>`, not `cockpit/<run-id>/<item.id>-<item.slug>`: git can't hold `cockpit/<run-id>` and `cockpit/<run-id>/…` together (beta's `runs.ts:1002`).
- **The review-step rule.** The contract gains: a non-interactive agent step or parallel group whose outcomes include `CHANGES` starts a fresh session each round, and its opening message names its earlier review file; every other step resumes its session.
- **The format's variables.** `skills/write-workflow/format.md:59` gets the same nine-variable list, dropping `{{project.path}}`, `{{<step>.outcome}}`, `{{<step>.files}}`, `{{<step>.answer}}`, `{{<step>.title}}`, `{{<step>.preface}}` and `{{item.slug}}`.
- **The land steps abort a stopped merge.** The `land` step of all four workflows (Add to backlog in `workflows/idea-2-backlog.yaml`, Merge in `backlog-2-shipped.yaml`, `bug-2-shipped.yaml` and `idea-2-shipped.yaml`) gains `on_stop: git merge --abort`, as the lane Merge already has.

The four workflows already use only beta's nine variables, so no workflow prompt or command changes beyond the land steps' `on_stop`.

## Acceptance criteria
- [ ] `docs/contract.md`'s Variables line names exactly the nine variables, none of `item.slug`, `<step>.title` or `<step>.preface`, and says any other `{{…}}` is a parse error naming the file, step and variable.
- [ ] `grep 'cockpit/<run-id>/' docs/contract.md` finds nothing, and the contract names `cockpit/<run-id>-<item.id>-<item.slug>`.
- [ ] The contract states the fresh-session rule for non-interactive agent steps and parallel groups with a `CHANGES` outcome, and that every other step resumes.
- [ ] `skills/write-workflow/format.md`'s Variables line names exactly the nine variables, none of `project.path`, `<step>.outcome`, `<step>.files`, `<step>.answer`, `<step>.title`, `<step>.preface` or `item.slug`.
- [ ] Each of the four workflows' `land` step has `on_stop: git merge --abort`.
- [ ] All four workflows still load.

## Verified by
A grep for each of the points above, and a test or dry parse of the four workflow files showing they still load.
