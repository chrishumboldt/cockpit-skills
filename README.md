# Cockpit Skills

**Flight plans for your coding agents.** The skills and workflows for [Cockpit](https://github.com/chrishumboldt/cockpit), the desktop app by Renegade Studios for running coding agents side by side.

Give it an idea or a bug. Your agents question it, plan it, build it test-first, review it and merge it, and you sign off at the moments that matter: the spec, the split into work, and the ship.

```
  an idea ──▶ Idea 2 Backlog ──▶ docs/backlog/ ──▶ Backlog 2 Shipped ──▶ merged
       └───────────────────── Idea 2 Shipped ───────────────────────┘

  a bug ────▶ Bug 2 Shipped ─────────────────────────────────────────▶ merged
```

No issue tracker and no pull requests. Work lives in your repo as a backlog, one Markdown file per item in `docs/backlog/`, and a run that ships merges into the branch it started from and pushes it.

## Quick start

1. **Install.** In Cockpit, open **Skills** or **Workflows** and click **Install Cockpit Skills**. Already have other packages? Use **Install from GitHub** with `https://github.com/chrishumboldt/cockpit-skills`.
2. **Crew it.** Cockpit asks which of your agents fill the four roles (below).
3. **Check the repo is airworthy.** In any session, ask "is this repo ready for Cockpit's workflows?". The `airworthy` skill finds your test commands, checks your branch takes a push, and proposes a short section for your `CLAUDE.md` or `AGENTS.md`.
4. **Fly.** Type `/idea-2-backlog`, `/idea-2-shipped` or `/bug-2-shipped` in a session and say what you want.

**Update** in the Skills view pulls new releases; **Remove** takes it all out again.

### What your repo needs

- **An `origin` you can push to.** Runs merge into the branch they started from and push it, so that branch can't require pull requests. If your `main` is protected, fly from a working branch (`beta`, `develop`).
- **Its check commands written down.** Agents run only the tests for what they touched, using the commands your agent docs name (or your manifest's and CI's). `airworthy` writes these for you.
- **Optional:** a glossary (`GLOSSARY.md`) and ADRs (`docs/adr/`). The skills use them when they exist, so specs and tests speak your terms.

## The crew

| Role | Does | Suggested |
|---|---|---|
| **planner** | Questions ideas, writes specs, splits work, diagnoses bugs | Opus, medium effort |
| **implementer** | Builds each ticket test-first and fixes what reviews find | Sonnet, medium effort |
| **reviewer** | Reviews every change for correctness | Opus, high effort |
| **smell-checker** | Reviews the finished branch for code smells | Sonnet, medium effort |

A cheap implementer works well when the tickets are specific and the reviewer is strong. Step up the implementer for work that designs new core shape.

## The workflows

Steps marked **you** are gates: you approve, ask for changes, or stop the run.

### Idea 2 Backlog

`/idea-2-backlog` and the idea. Plans the work; builds nothing.

1. **Design**: asks once whether you're working from a design, and studies it if so.
2. **Preflight**: questions you one decision at a time, after reading your docs, code and backlog so it never asks what they already answer.
3. **Write spec**: turns the decisions into a spec.
4. **Review spec**: **you**.
5. **Write backlog items**: splits the spec into small end-to-end items, each naming what blocks it (`blockedBy`) and what else it needs (`needs`).
6. **Approve items**: **you**.
7. **Add to backlog**: merges the items into your branch and pushes it.

Small, clear things don't need a workflow: ask any session to add them, and `backlog-add` files the item.

### Backlog 2 Shipped

`/backlog-2-shipped`. Builds what's ready.

1. **Pick items**: lists the ready items and asks which to build, flagging blockers and stale items.
2. **Implement**: each item is built test-first on its own branch, reviewed, fixed if needed and merged into the run's branch. Items that don't block each other build at the same time.
3. **Final review**: the whole branch, for correctness and for smells.
4. **Fix**: what Final review or you asked for, then Final review looks again.
5. **Ship**: **you**.
6. **Merge**: merges into your branch, marks the items `done` and pushes.

### Idea 2 Shipped

`/idea-2-shipped` and the idea. Idea 2 Backlog's first six steps, then Backlog 2 Shipped's build on exactly those items: from an idea to merged code in one run, with the items in the backlog marked `done` as its record.

### Bug 2 Shipped

`/bug-2-shipped` and the bug. Proves it before it fixes it.

1. **Diagnose**: asks how to make it happen, reproduces it with a failing test, finds the root cause and commits the test. If it can't reproduce the bug, it says what would help and stops rather than guessing.
2. **Implement**: makes that test pass, at the cause.
3. **Code review**: checks the cause is gone, not just the symptom, and that the test failed before the fix.
4. **Ship**: **you**.
5. **Merge**: merges the fix into your branch and pushes.

Want more in a run, such as your full test suites or screenshots before shipping? Ask any session to adapt a workflow with `write-workflow`.

## The skills

Each skill does one step's job. Three are for you to call from the chat.

| | Skill | What it does |
|---|---|---|
| **Plan** | `design-intake` | Asks once for a design and writes down what it shows. |
| | `preflight` | Questions an idea until it's settled. |
| | `write-spec` | Turns the decisions into a spec. |
| | `to-tickets` | Splits a spec into tickets for a run. |
| | `backlog-items` | Splits a spec into backlog items. |
| | `backlog-pick` | Asks which ready items to build and turns them into tickets. |
| **Build** | `implement` | Builds one ticket test-first, or fixes a diagnosed bug. |
| | `diagnose` | Reproduces a bug with a failing test and finds its root cause. |
| | `resolve-merge` | Re-merges a lane after a conflict, keeping both sides. |
| **Review** | `code-review` | Reviews a change for correctness, or for smells only. |
| **Chat** | `airworthy` | Checks a repo is ready to fly the workflows, once. |
| | `backlog-add` | Adds an item to the backlog, no workflow needed. |
| | `write-workflow` | Writes a new workflow with you, or adapts one of these. |

Cockpit itself teaches your home agent to start and follow runs, so there's nothing to install for that.

## Making it yours

It's all plain Markdown and YAML, and Cockpit shows you each skill's text before it runs.

- **Tweak a skill:** open it in Cockpit and choose **Duplicate to edit** for your own copy in your local package.
- **Fork the package:** edit the skills or workflows in your fork and install that instead. Give it its own `name:` in `cockpit.yaml` so it can sit beside this one.
- **Contribute back:** pull requests welcome. Keep to the same plain, concrete style and Cockpit's terms (agent, session, run, step, project, workflow, role), and keep [docs/contract.md](docs/contract.md), which says which files each step reads and writes, in step with the workflows.

## Licence

MIT, Copyright (c) 2026 Renegade Studios. The preflight, write-spec, to-tickets, implement and code-review skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills), Copyright (c) 2026 Matt Pocock, MIT. His notice is in [LICENSE](LICENSE).
