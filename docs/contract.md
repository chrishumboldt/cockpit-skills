# How the skills and workflows fit together

The reference for anyone changing a skill or a workflow: how Cockpit runs a step, which files each step reads and writes, and the outcome words it reports. The friendly overview is in the [README](../README.md).

## How a step runs a skill

- **Invocation.** The step's opening message is the skill's slash command, `/cockpit:<skill>`, then the step's resolved `prompt:`, then (for a step with `context: { transcript: <step> }`) "The conversation so far is in `<file>`; read it first.", then the step contract footer naming the step number, the run folder and the outcomes.
- **The run folder** is `.cockpit/runs/<run-id>/` in the project. `reads`, `writes` and `shows` are relative to it; the prompts name it as `{{run.folder}}`.
- **Outcomes.** A skill calls `report_outcome` (`mcp__cockpit__report_outcome`) with one of its step's declared outcomes and a one-line summary, or with a summary alone when the step declares none. Without the call, the engine reads the first line of the step's first written file (`OUTCOME: PASS`), which is why review files start with that line.
- **Artifacts.** `write-spec`, `to-tickets` and `code-review` also call `submit_artifact` with kind `spec`, `tickets` or `review`.
- **Back from a gate.** Request changes resumes the step's session with the notes and writes them to `gate-<step>.md` in the run folder; the skill revises its files in place. A step the gate sends to fix what it found in another step's file (Fix, from Ship) gets them in the gate's own file instead, `gate-ship.md`, and starts with them in its opening message if it hasn't run yet.
- **The lane folder.** Inside the fan-out each ticket runs in its own lane, with a lane folder `lanes/<item.id>/` in the run folder, named `{{lane.folder}}`. A bare `writes:` entry in a lane lands in the lane folder; a bare `reads:` entry looks in the lane folder first, then in the run folder. So a lane's `reads: [spec.md, 'tickets/*.md', 'review-*.md']` finds the run's spec and tickets and the lane's own reviews, and a lane never overwrites `spec.md`. A lane step's `gate-<step>.md`, `transcript-<step>.md` and `<step>.log` also go in the lane folder.
- **Variables.** Besides `{{run.folder}}`, `{{run.branch}}` (the run's integration branch), `{{run.base}}` (the branch the run was cut from, which the pull request targets) and `{{<step>.file}}`, `{{<step>.title}}` (the step's first file's first heading, else the run's branch, with ` (#N)` after it when that file has an `Issue: #N` line in its head) and `{{<step>.preface}}` (the paragraphs a pull request's description opens with: `Closes #N` when the file names an issue, then `Opened by Cockpit run <id> (<workflow>).`, each its own word in a command), a lane's prompts and commands take `{{item.id}}`, `{{item.slug}}`, `{{item.title}}` and `{{item.file}}` (from `tickets.json`), `{{item.branch}}` (the lane's ticket branch, `cockpit/<run-id>/<item.id>-<item.slug>`, made from the integration branch when the lane starts) and `{{lane.folder}}`. For a step that works outside the run folder's root, `{{run.folder}}`, `{{lane.folder}}` and the step file variables print as absolute paths.
- **`run:` steps.** A `run:` step is a shell command Cockpit runs itself, in the run worktree, through the user's login shell, so `git` and `gh` are the login shell's own, with its sign-in. Exit 0 is `PASS`, any other exit `FAIL`; an unrouted `FAIL` opens the step's failure gate. Its output goes to `<step key>.log` (in the lane folder inside a lane). `on_stop:` is a command run first when the owner stops the lane or run from that failure gate; its exit code is ignored.

## File contract

Paths are in the run folder unless they start with `/` (the project folder). "Engine" files are written by Cockpit, not by a skill.

| Step (id) | Skill | Reads | Writes | Outcomes |
|---|---|---|---|---|
| Design (`design`) | `design-intake` | `transcript-design.md` (engine: the home chat, when passed) | `design.md`, always, a one-liner when there is no design | `DESIGN`, `NONE` |
| Preflight (`preflight`) | `preflight` | `transcript-design.md` (engine), `design.md`, the repo's glossary, ADRs, spec and code | `preflight-notes.md`: the idea as settled, decisions with reasons, terms, facts from the repo, possible ADRs, open questions | none (summary only) |
| Write spec (`spec`) | `write-spec` | `preflight-notes.md`, `transcript-preflight.md` (engine), `transcript-design.md` (engine), `design.md`, `gate-spec.md` (engine, on a revision) | `spec.md`: a title, an `Issue: #N` line when the idea builds a GitHub issue, then Goal, Non-goals, Behaviour, Decisions, Testing, Acceptance, Open questions | none (summary only) |
| Review spec (`review-spec`) | gate | shows `spec.md` | `gate-spec.md` (engine, on Request changes) | `APPROVE`, `CHANGES`, `REJECT` |
| Implement › Implement (`implement`, per ticket) | `implement` | `spec.md`, the ticket's `tickets/NN-<slug>.md` (both from the run folder), `review-*.md` (the lane's) | lane folder: `build.md`: Built, Verified, Review findings, Left; commits on `{{item.branch}}` | none (summary only) |
| Implement › Code review | `code-review`, correctness | the ticket file, `spec.md`, the lane's diff | lane folder: `review-code.md` | `PASS`, `CHANGES` |
| Implement › Try merge (`try-merge`, per ticket) | `run:` `git merge --no-ff {{item.branch}}`; aborts the merge, leaving the worktree clean, on a conflict or a numbered-file clash (the rule is in the script) | | lane folder: the step's log (engine) | `PASS` (to Merge resolved lane), `FAIL` (to Resolve merge) |
| Implement › Resolve merge (`resolve`, per ticket, only after a conflict) | `resolve-merge` | `spec.md`, `tickets/*.md` | lane folder: `resolve.md`; the merge commit on the lane branch (the run branch merged into it, in the lane's worktree) | `DONE` |
| Implement › Merge resolved lane (`merge`, per ticket) | `run:` the same merge, the same script, which is a no-op after Try merge and merges the resolved lane after Resolve merge; it counts the ticket merged. `on_stop: git merge --abort` | | lane folder: the step's log (engine) | `PASS`, `FAIL` |
| Final review › Code review | `code-review`, correctness | `spec.md`, the integration branch's diff | `final-review-code.md` | `PASS`, `CHANGES` |
| Final review › Code smell | `code-review`, smells only | the integration branch's diff | `final-review-smell.md` | `PASS`, `CHANGES` |
| Fix (`fix`) | `implement` | `final-review-*.md`, `spec.md`, `gate-ship.md` (engine, after Ship's Request changes) | `fix.md`: what it changed and why; commits on the integration branch | `DONE` |
| Ship (`ship`) | gate | shows `final-review-code.md` | `gate-ship.md` (engine, on Request changes) | `APPROVE`, `CHANGES`, `REJECT` |

**In Idea 2 Shipped** Write backlog items and Approve items come after Review spec; Write backlog items also writes the run's tickets (`picks.txt`, `tickets.md`, `tickets.json`, `tickets/*.md`), which the fan-out builds. There is no Pick step.

**Routes after the fan-out.** Final review's `PASS` skips to Ship and `CHANGES` goes to Fix. Fix's `DONE` goes back to Final review, at most three times. Ship's Request changes goes to Fix with the notes in `gate-ship.md`; Approve runs Merge; Reject ends the run. A failed Merge opens its failure gate. A conflict in a lane's merge goes to Resolve merge first; when it can't, it reports `DONE` with a summary starting "Couldn't resolve", Merge conflicts again and fails, and the lane's failure gate opens with Fix it myself.

**`tickets.json`** is a JSON array in build order, blockers first:

```json
[
  { "id": "01", "slug": "pause-on-failure", "title": "Pause the run when a step fails", "blockedBy": [], "file": "tickets/01-pause-on-failure.md" },
  { "id": "02", "slug": "resume-a-paused-run", "title": "Resume a paused run", "blockedBy": ["01"], "file": "tickets/02-resume-a-paused-run.md" }
]
```

`id` is two digits; `blockedBy` names earlier ids only; `file` is relative to the run folder. Idea 2 Shipped's fan-out (`foreach: tickets.frontier`) runs these, and its prompts use `{{item.id}}`, `{{item.title}}` and `{{item.file}}` from them, with `{{item.branch}}` and `{{lane.folder}}` from Cockpit. A lane's own files (`build.md`, `review-*.md`) sit in its lane folder, `lanes/<item.id>/`.

**A review file** starts with the line `OUTCOME: PASS` or `OUTCOME: CHANGES`, then its findings under Changes (must change) and Optional, each as `file:line`, the problem and what to change.

## The backlog workflows

`Idea 2 Backlog` and `Backlog 2 Shipped` share the steps above where they have the same keys, and add these. The backlog's format is `skills/backlog-items/format.md`; items live in `docs/backlog/` in the project, committed like code.

| Step (id) | Skill | Reads | Writes | Outcomes |
|---|---|---|---|---|
| Write backlog items (`items`, Idea 2 Backlog) | `backlog-items` | `spec.md`, `docs/backlog/`, `gate-items.md` (engine, on a revision) | `docs/backlog/NNN-<slug>.md` and `docs/backlog/specs/<slug>.md`, committed on the run's branch; `items.md` (the index the gate shows) | `READY` |
| Approve items (`approve-items`) | gate | shows `items.md` | `gate-items.md` (engine, on Request changes) | `APPROVE`, `CHANGES`, `REJECT` |
| Add to backlog (`land`, Idea 2 Backlog) | `run:` merge the run's branch into `{{run.base}}` (`--no-ff`, in the run worktree) and push it | | the step's log (engine) | `PASS`, `FAIL` |
| Pick items (`pick`, Backlog 2 Shipped) | `backlog-pick`, interactive | `docs/backlog/` | `picks.txt` (the items' repo paths), `spec.md`, `tickets.md`, `tickets.json`, `tickets/*.md`; a commit on the run's branch for each item the user marks `idea` or `done` | `READY`, `NONE` (ends the run) |
| Ship (`ship`, Backlog 2 Shipped) | gate | shows `final-review-code.md` | `gate-ship.md` (engine, on Request changes, read by Fix) | `APPROVE`, `CHANGES`, `REJECT` |
| Merge (`land`, Backlog 2 Shipped) | `run:` sets the picked items to `status: done` with a `done:` date, commits, merges into `{{run.base}}` (`--no-ff`) and pushes it | `picks.txt` | the step's log (engine) | `PASS`, `FAIL` |

**Your own steps.** A project that wants more (its full test suites, screenshots to look at) copies Backlog 2 Shipped into its local package and adds `run:` steps there.

## Known limit: the merge script is copied

The Try merge and Merge resolved lane `run:` scripts are byte-identical, in both shipped workflows (four copies). A `run:` executes in the run folder, not in this repo, so it can't call a script file kept here, and Merge can't loop back to Try merge: `end` and shell-step loops aren't allowed inside a fan-out (`skills/write-workflow/format.md`). Until the engine allows either, change all four copies together.
