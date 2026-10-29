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

## The workflows

Two pairs. **Idea 2 Tickets** and **Idea 2 Shipped** take one idea now, through GitHub. **Idea 2 Backlog** and **Backlog 2 Shipped** keep a backlog in your repo instead: you add to it as you go and build from it in batches, with no issue tracker and no pull requests.

### Idea 2 Tickets

Start it from any session with `/idea-2-tickets` and tell it what you want to build.

1. **Design.** It asks once whether the work comes from a design (a link or an image) and studies it if so.
2. **Preflight.** It questions you about the idea, one question at a time, until the decisions are made. It checks your repo's docs and code first so it never asks what the codebase already answers.
3. **Write spec.** It writes a spec from those decisions.
4. **Review spec.** You read the spec and approve it, ask for changes, or stop.
5. **Break into tickets.** It splits the spec into small, end-to-end tickets, each saying what blocks it.
6. **Approve tickets.** You read the breakdown and approve it, ask for changes, or stop.

Everything it writes goes into your project, under `.cockpit/runs/<run>/`: the design notes, the preflight notes, the spec, and one file per ticket. Nothing is pushed to GitHub or any issue tracker.

### Idea 2 Shipped

The same six steps, then it builds the tickets:

7. **Implement.** Each ticket is built test-first on its own branch, reviewed for correctness and for code smells, sent back for fixes if needed, and merged.
8. **Final review.** The whole result is reviewed once more.
9. **Fix.** When Final review or you ask for changes, an agent fixes them and Final review looks again (up to twice).
10. **Ship.** You approve, which opens the pull request and merges it, or ask for changes, which go to Fix.
11. **Open pull request.** It pushes the branch and opens the pull request, titled by the spec's heading and its issue, with the spec as its description. When the spec names the GitHub issue it builds (`Issue: #418`), the title ends with `(#418)` and the description opens with `Closes #418`.
12. **Merge pull request.** It merges the pull request into the branch the run started from, as a merge commit, and deletes the pushed branch. A merge that fails (a conflict, a required check) stops the run at that step, with the pull request left open.

Idea 2 Shipped needs a version of Cockpit that can run steps in parallel. Until then Cockpit lists it but won't start it.

### Idea 2 Backlog

For an idea that needs thinking before it can be built. The first four steps are Idea 2 Tickets' (Design, Preflight, Write spec, Review spec; Preflight also reads the backlog for overlap), then:

5. **Write backlog items.** It splits the spec into items in `docs/backlog/`, each naming the items that block it (`blockedBy`) and anything else it needs (`needs`), and saves the spec beside them.
6. **Approve items.** You read the split and approve it, ask for changes, or stop.
7. **Add to backlog.** It merges the items into the branch the run started from and pushes it.

Small, clear things don't need a workflow: ask any session to add them, and the `backlog-add` skill writes the item and pushes it.

### Backlog 2 Shipped

1. **Pick items.** It lists the ready items and asks which to build, checking blockers and `needs` with you.
2. **Implement.** Each item is built on its own branch, reviewed for correctness and code smells, and merged into the run's branch; items that don't block each other build at the same time.
3. **Final review**, and **Fix** when it or a later step asks for changes.
4. **Ship.** You approve, or ask for changes, which go to Fix.
5. **Merge.** It marks the items `done`, merges the run's branch into the branch the run started from as one merge commit, and pushes it.

The backlog's format is in [skills/backlog-items/format.md](skills/backlog-items/format.md); the first item written copies it to `docs/backlog/README.md`.

## The skills

Each skill does one step's job. The workflows call them; only `backlog-add` is meant for the chat.

| Skill | What it does |
|---|---|
| `design-intake` | Asks once for a design and writes down what it finds. |
| `preflight` | Questions the idea until it's settled and writes the notes. |
| `write-spec` | Turns the notes into a spec. |
| `to-tickets` | Splits the spec into tickets, written as files. |
| `implement` | Builds one ticket, test-first, and reports what it did. |
| `code-review` | Reviews a change, for correctness or for code smells only. |
| `backlog-items` | Splits a spec into backlog items. |
| `backlog-pick` | Asks which ready backlog items to build and turns them into the run's tickets. |
| `backlog-add` | Adds an item to the backlog from the chat, with no workflow. You can call this one yourself. |

Cockpit itself teaches your home agent how to start and follow runs, so there's nothing to install for that.

## Making it yours

Everything here is plain Markdown and YAML, and Cockpit shows you each skill's text before it runs.

- **Tweak a skill:** in Cockpit, open the skill and choose **Duplicate to edit** to get your own copy in your local package.
- **Change the package itself:** fork this repo, edit the `SKILL.md` files or the workflows, and install your fork instead. Give it a different `name:` in `cockpit.yaml` so it can live beside this one.
- **Contribute back:** pull requests are welcome. Write skills in the same plain, concrete style, using Cockpit's terms (agent, session, run, step, project, workflow, role), and keep the file contract in [docs/contract.md](docs/contract.md) in step with the workflows.

## Licence

MIT, Copyright (c) 2026 Renegade Studios. The preflight, write-spec, to-tickets, implement and code-review skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills), Copyright (c) 2026 Matt Pocock, MIT. His notice is in [LICENSE](LICENSE).
