# How the skills and workflows fit together

The reference for anyone changing a skill or a workflow: how Cockpit runs a step, which files each step reads and writes, and the outcome words it reports. The friendly overview is in the [README](../README.md).

## How a step runs a skill

- **Invocation.** The step's opening message is the skill's slash command, `/cockpit:<skill>`, then the step's resolved `prompt:`, then (for a step with `context: { transcript: <step> }`) "The conversation so far is in `<file>`; read it first.", then the step contract footer naming the step number, the run folder and the outcomes.
- **The run folder** is `.cockpit/runs/<run-id>/` in the project. `reads`, `writes` and `shows` are relative to it; the prompts name it as `{{run.folder}}`.
- **Outcomes.** A skill calls `report_outcome` (`mcp__cockpit__report_outcome`) with one of its step's declared outcomes and a one-line summary, or with a summary alone when the step declares none. Without the call, the engine reads the first line of the step's first written file (`OUTCOME: PASS`), which is why review files start with that line.
- **Artifacts.** `write-spec`, `to-tickets` and `code-review` also call `submit_artifact` with kind `spec`, `tickets` or `review`.
- **Back from a gate.** Request changes resumes the step's session with the notes and writes them to `gate-<step>.md` in the run folder; the skill revises its files in place.
- **The lane folder.** Inside the fan-out each ticket runs in its own lane, with a lane folder `lanes/<item.id>/` in the run folder, named `{{lane.folder}}`. A bare `writes:` entry in a lane lands in the lane folder; a bare `reads:` entry looks in the lane folder first, then in the run folder. So a lane's `reads: [spec.md, 'tickets/*.md', 'review-*.md']` finds the run's spec and tickets and the lane's own reviews, and a lane never overwrites `spec.md`. A lane step's `gate-<step>.md`, `transcript-<step>.md` and `<step>.log` also go in the lane folder.
- **Variables.** Besides `{{run.folder}}`, `{{run.branch}}` (the run's integration branch) and `{{<step>.file}}`, a lane's prompts and commands take `{{item.id}}`, `{{item.slug}}`, `{{item.title}}` and `{{item.file}}` (from `tickets.json`), `{{item.branch}}` (the lane's ticket branch, `cockpit/<run-id>/<item.id>-<item.slug>`, made from the integration branch when the lane starts) and `{{lane.folder}}`. For a step that works outside the run folder's root, `{{run.folder}}`, `{{lane.folder}}` and the step file variables print as absolute paths.
- **`run:` steps.** A `run:` step is a shell command Cockpit runs itself, in the run worktree, through the user's login shell, so `git` and `gh` are the login shell's own, with its sign-in. Exit 0 is `PASS`, any other exit `FAIL`; an unrouted `FAIL` opens the step's failure gate. Its output goes to `<step key>.log` (in the lane folder inside a lane). `on_stop:` is a command run first when the owner stops the lane or run from that failure gate; its exit code is ignored.

## File contract

Paths are in the run folder unless they start with `/` (the project folder). "Engine" files are written by Cockpit, not by a skill.

| Step (id) | Skill | Reads | Writes | Outcomes |
|---|---|---|---|---|
| Design (`design`) | `design-intake` | `transcript-design.md` (engine: the home chat, when passed) | `design.md`, always, a one-liner when there is no design | `DESIGN`, `NONE` |
| Preflight (`preflight`) | `preflight` | `transcript-design.md` (engine), `design.md`, the repo's glossary, ADRs, spec and code | `preflight-notes.md`: the idea as settled, decisions with reasons, terms, facts from the repo, possible ADRs, open questions | none (summary only) |
| Write spec (`spec`) | `write-spec` | `preflight-notes.md`, `transcript-preflight.md` (engine), `transcript-design.md` (engine), `design.md`, `gate-spec.md` (engine, on a revision) | `spec.md`: Goal, Non-goals, Behaviour, Decisions, Testing, Acceptance, Open questions | none (summary only) |
| Review spec (`review-spec`) | gate | shows `spec.md` | `gate-spec.md` (engine, on Request changes) | `APPROVE`, `CHANGES`, `REJECT` |
| Break into tickets (`tickets`) | `to-tickets` | `spec.md`, `gate-tickets.md` (engine, on a revision) | `tickets/NN-<slug>.md` (one per ticket, from `01` in dependency order: title, Blocked by, What to build, Acceptance criteria, Verified by); `tickets.md` (the human index: per ticket its number, title, Blocked by, what it delivers, file); `tickets.json` (the machine index, below) | `READY` |
| Approve tickets (`approve-tickets`) | gate | shows `tickets.md` | `gate-tickets.md` (engine, on Request changes) | `APPROVE`, `CHANGES`, `REJECT` |
| Implement › Implement (`implement`, per ticket) | `implement` | `spec.md`, the ticket's `tickets/NN-<slug>.md` (both from the run folder), `review-*.md` (the lane's) | lane folder: `build.md`: Built, Verified, Review findings, Left; commits on `{{item.branch}}` | none (summary only) |
| Implement › Review › Code review | `code-review`, correctness | the ticket file, `spec.md`, the lane's diff | lane folder: `review-code.md` | `PASS`, `CHANGES` |
| Implement › Review › Code smell | `code-review`, smells only | the lane's diff | lane folder: `review-smell.md` | `PASS`, `CHANGES` |
| Implement › Merge (`merge`, per ticket) | `run:` `git merge --no-ff {{item.branch}}`, `on_stop: git merge --abort` | | lane folder: the step's log (engine) | `PASS`, `FAIL` |
| Final review › Code review | `code-review`, correctness | `spec.md`, the integration branch's diff | `final-review-code.md` | `PASS`, `CHANGES` |
| Final review › Code smell | `code-review`, smells only | the integration branch's diff | `final-review-smell.md` | `PASS`, `CHANGES` |
| Fix (`fix`) | `implement` | `final-review-*.md`, `spec.md`, `gate-ship.md` (engine, after Ship's Request changes) | `fix.md`: what it changed and why; commits on the integration branch | `DONE` |
| Ship (`ship`) | gate | shows `final-review-code.md` | `gate-ship.md` (engine, on Request changes) | `APPROVE`, `CHANGES`, `REJECT` |
| Open pull request (`publish`) | `run:` `git push` and `gh pr create`, the body from `spec.md` | | the step's log (engine); the pull request's URL is its last line | `PASS`, `FAIL` |

**Routes after the fan-out.** Final review's `PASS` skips to Ship and `CHANGES` goes to Fix. Fix's `DONE` goes back to Final review, at most twice. Ship's Request changes goes to Fix with the notes in `gate-ship.md`; Approve runs Open pull request; Reject ends the run.

**`tickets.json`** is a JSON array in build order, blockers first:

```json
[
  { "id": "01", "slug": "pause-on-failure", "title": "Pause the run when a step fails", "blockedBy": [], "file": "tickets/01-pause-on-failure.md" },
  { "id": "02", "slug": "resume-a-paused-run", "title": "Resume a paused run", "blockedBy": ["01"], "file": "tickets/02-resume-a-paused-run.md" }
]
```

`id` is two digits; `blockedBy` names earlier ids only; `file` is relative to the run folder. Idea → Shipped's fan-out (`foreach: tickets.frontier`) runs these, and its prompts use `{{item.id}}`, `{{item.title}}` and `{{item.file}}` from them, with `{{item.branch}}` and `{{lane.folder}}` from Cockpit. A lane's own files (`build.md`, `review-*.md`) sit in its lane folder, `lanes/<item.id>/`.

**A review file** starts with the line `OUTCOME: PASS` or `OUTCOME: CHANGES`, then its findings under Changes (must change) and Optional, each as `file:line`, the problem and what to change.
