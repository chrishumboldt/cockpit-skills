# Cockpit Skills

Skills and workflows for [Cockpit](https://github.com/chrishumboldt/renegade-cockpit), the desktop app by Renegade Studios for running coding agents side by side.

Install this package and your agents can take an idea all the way to a set of tickets, or all the way to shipped code, with you approving the important bits along the way.

## Installing

Open Cockpit and go to **Skills** or **Workflows**. If you have nothing installed yet, you'll see an **Install Cockpit Skills** button. Click it and you're done.

Already have other packages? Use **Install from GitHub** and paste this repo's URL:

```
https://github.com/chrishumboldt/cockpit-skills
```

Cockpit asks you which of your agents fill the package's roles (a planner, an implementer, a reviewer and a smell-checker), and then the workflows show up in your session's `/` menu. **Update** in the Skills view pulls new releases; **Remove** takes it all out again.

## The three workflows

Work lives in a **backlog** in your repo, `docs/backlog/`, one Markdown file per item ([the format](skills/backlog-items/format.md)). No issue tracker and no pull requests: a run that ships merges into the branch it started from and pushes it.

### Idea 2 Backlog

Start it from any session with `/idea-2-backlog` and tell it the idea.

1. **Design.** It asks once whether the work comes from a design (a link or an image) and studies it if so.
2. **Preflight.** It questions you about the idea, one question at a time, until the decisions are made. It checks your repo's docs, code and backlog first, so it never asks what they already answer.
3. **Write spec.** It writes a spec from those decisions.
4. **Review spec.** You read the spec and approve it, ask for changes, or stop.
5. **Write backlog items.** It splits the spec into small, end-to-end items in `docs/backlog/`, each naming the items that block it (`blockedBy`) and anything else it needs (`needs`), and saves the spec beside them.
6. **Approve items.** You read the split and approve it, ask for changes, or stop.
7. **Add to backlog.** It merges the items into the branch the run started from and pushes it.

Small, clear things don't need a workflow: ask any session to add them, and the `backlog-add` skill writes the item and pushes it.

### Backlog 2 Shipped

1. **Pick items.** It lists the ready items and asks which to build, checking blockers and `needs` with you.
2. **Implement.** Each item is built test-first on its own branch, reviewed for correctness, sent back for fixes if needed, and merged into the run's branch. Items that don't block each other build at the same time.
3. **Final review.** The whole result is reviewed once more.
4. **Fix.** When Final review or you ask for changes, an agent fixes them and Final review looks again.
5. **Ship.** You approve, or ask for changes, which go to Fix.
6. **Merge.** It marks the items `done`, merges the run's branch into the branch the run started from as one merge commit, and pushes it.

### Idea 2 Shipped

Idea 2 Backlog's first six steps (its Write backlog items also writes the run's tickets), then Backlog 2 Shipped's build on exactly those items, in one run, with nothing to pick: from an idea to merged code, with the items in the backlog marked `done` as its record. Start it with `/idea-2-shipped` and the idea.

Want more in a run, such as your full test suites or screenshots to look at before shipping? Ask any session to adapt one with the `write-workflow` skill: it copies a workflow into your local package and adds `run:` steps there.

## The skills

Each skill does one step's job. The workflows call them; `backlog-add` and `write-workflow` are meant for the chat.

| Skill | What it does |
|---|---|
| `design-intake` | Asks once for a design and writes down what it finds. |
| `preflight` | Questions the idea until it's settled and writes the notes. |
| `write-spec` | Turns the notes into a spec. |
| `to-tickets` | Splits a spec into tickets in the run folder; `backlog-items` and `backlog-pick` follow its slicing and its `tickets.json`. |
| `implement` | Builds one ticket, test-first, and reports what it did. |
| `resolve-merge` | Merges a lane again after a conflict, keeps both sides, renumbers clashing ADRs and commits. |
| `code-review` | Reviews a change, for correctness or for code smells only. |
| `backlog-items` | Splits a spec into backlog items. |
| `backlog-pick` | Asks which ready backlog items to build and turns them into the run's tickets. |
| `backlog-add` | Adds an item to the backlog from the chat, with no workflow. You can call this one yourself. |
| `write-workflow` | Designs and writes a workflow with you, new or adapted from one of these, for your local package or your own. You can call this one yourself. |

Cockpit itself teaches your home agent how to start and follow runs, so there's nothing to install for that.

## Making it yours

Everything here is plain Markdown and YAML, and Cockpit shows you each skill's text before it runs.

- **Tweak a skill:** in Cockpit, open the skill and choose **Duplicate to edit** to get your own copy in your local package.
- **Change the package itself:** fork this repo, edit the `SKILL.md` files or the workflows, and install your fork instead. Give it a different `name:` in `cockpit.yaml` so it can live beside this one.
- **Contribute back:** pull requests are welcome. Write skills in the same plain, concrete style, using Cockpit's terms (agent, session, run, step, project, workflow, role), and keep the file contract in [docs/contract.md](docs/contract.md) in step with the workflows.

## Licence

MIT, Copyright (c) 2026 Renegade Studios. The preflight, write-spec, to-tickets, implement and code-review skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills), Copyright (c) 2026 Matt Pocock, MIT. His notice is in [LICENSE](LICENSE).
