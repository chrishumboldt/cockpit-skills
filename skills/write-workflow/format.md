# The workflow file format

A workflow is one YAML file: `name:`, `description:` (both optional) and `steps:`, a mapping of step keys in run order. Unknown keys are errors. The app reports the first error it finds, with its line.

```yaml
name: Quick fix
description: Implement a small change, review it, and merge it.

steps:
  implement:
    name: Implement
    agent: Grunt                 # local workflows; a package uses role: implementer
    skill: cockpit/implement
    prompt: Implement what `{{run.folder}}/task.md` asks; write what you built to `{{run.folder}}/build.md`.
    reads: [task.md]
    writes: [build.md]

  review:
    name: Review
    parallel:
      code:  { agent: Hard Worker, skill: cockpit/code-review, prompt: Review for correctness; write review-code.md. }
      smell: { agent: Grunt, skill: cockpit/code-review, prompt: Review for code smells only; write review-smell.md. }
    writes: ['review-*.md']
    outcomes: [PASS, CHANGES]
    on:
      CHANGES: { to: implement, max: 2 }

  ship:
    name: Ship
    gate: approval
    shows: build.md
    on:
      CHANGES: implement
      REJECT: end

  # Merges the run's branch into the base and pushes it.
  land:
    name: Merge
    run: sh -c 'git fetch -q origin "$2" && git checkout -q --detach "origin/$2" && git merge --no-ff --no-edit "$1" && git push -q origin "HEAD:$2"' sh {{run.branch}} {{run.base}}
```

## Order and routing
- Steps run in document order. Each flows to the next unless its outcome is routed.
- `on:` maps an outcome to a later step (a skip) or an earlier one (a loop). A loop from an agent step or a group needs `max:` (`{ to: build, max: 2 }`); a route out of a gate doesn't. Hitting `max` raises a gate.
- `end` finishes the run there (`REJECT: end`). Not inside a fan-out.

## Step types (the keys decide)
- **Agent step:** `role:` or `agent:`, plus `skill:`. Optional: `name`, `prompt` (a template), `context` (`files` by default; `{ transcript: <step> }` passes that step's conversation; `{ continue: <step> }` resumes its session), `reads`, `writes`, `outcomes` (best to worst), `on`, `interactive` (true for a step that talks with the user), `pause`, `permissions`.
- **Parallel group:** `parallel:` with child agent steps inline. The group, not the children, carries `writes`, `outcomes` and `on`; it reports its worst child's outcome.
- **Fan-out:** `foreach: tickets.frontier` with child steps under `steps:`. It reads `tickets.json` in the run folder; each ticket becomes a lane with its own branch (`{{item.branch}}`), worktree and lane folder, and starts when its `blockedBy` tickets have merged. The child keyed `merge` counts the ticket as merged. A child may be a parallel group, nothing deeper.
- **Gate:** `gate: approval` (shows a file with `shows:`; outcomes APPROVE, CHANGES, REJECT; Request changes writes `gate-<step>.md` and resumes the step it routes to), `gate: answer` (asks `prompt:`, keeps the text in `{{<step>.answer}}`), `gate: choice` (`choices:`, each an outcome).
- **Shell step:** `run:` a command the app runs in the run worktree through the user's login shell. Exit 0 is the first outcome (`PASS`), anything else the last (`FAIL`); an unrouted FAIL raises a failure gate. Optional `on_stop:` runs when it's stopped. Output goes to `<step>.log`.

## Files
- `reads`, `writes` and `shows` are relative to the run folder (`.cockpit/runs/<run-id>/` in the project). An entry starting with `/` is a repo path (`/docs/adr/*.md`). Globs are fine; `..` is an error.
- Inside a fan-out a bare `writes` entry lands in the lane folder, and a bare `reads` entry looks there first, then in the run folder.

## Variables (a closed list)
`{{run.folder}}`, `{{run.branch}}` (the integration branch), `{{run.base}}` (the branch the run started from), `{{project.path}}`, `{{<step>.outcome}}`, `{{<step>.file}}` (its first written file), `{{<step>.files}}`, `{{<step>.answer}}`, `{{<step>.title}}`, `{{<step>.preface}}`; inside a fan-out `{{item.id}}`, `{{item.slug}}`, `{{item.title}}`, `{{item.file}}`, `{{item.branch}}` and `{{lane.folder}}`. In a `run:` command each variable is quoted as one shell word. Step keys are a letter then letters, digits, `-` or `_`; `run`, `project`, `item`, `lane` and `end` are reserved.

## Skills a step can use
Any installed skill, by `<package>/<skill>`. The `cockpit` package has `design-intake`, `preflight`, `write-spec`, `to-tickets`, `backlog-items`, `backlog-pick`, `implement`, `resolve-merge`, `code-review` (correctness, or smells only when the prompt says so) and, for chat, `backlog-add` and this one. A skill reports its outcome with `report_outcome`; review skills also start their file with `OUTCOME: PASS` or `OUTCOME: CHANGES`.
