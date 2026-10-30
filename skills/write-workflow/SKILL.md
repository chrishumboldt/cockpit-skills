---
name: write-workflow
description: Design and write a Cockpit workflow (YAML) with the user — a new one, or a copy of a packaged one changed to fit a project (extra checks, screenshots, other agents). Use when asked to "make a workflow", "build a workflow for us", "add a step to our workflow" or to change how runs go.
disable-model-invocation: false
---

# Write a workflow

You help the user turn how they actually work into a Cockpit workflow: a fixed series of steps the app runs, each step an agent with a skill, a gate the user answers, a shell command, a parallel group or a fan-out. The file format is in `format.md` beside this skill; read it before writing anything.

## 1. Understand the process first
Ask, one question at a time, only what you can't read from the repo:
- What goes in (an idea, backlog items, a ticket), and what comes out (a spec, items, merged code)?
- Where must a person decide? Those are gates. Keep them few: each one stops the run until the user answers.
- What does the project need that a generic workflow doesn't do? Its own test commands, screenshots, a lock on heavy jobs, a deploy. Those are `run:` steps, written straight into the workflow, not conventions other projects must follow.
- Which agent should do which step, given cost? Strong models where judgment matters (preflight, spec, final correctness review), cheaper ones for building and smell checks.

Start from an existing workflow when one is close: read the installed packages' `workflows/*.yaml` (the `cockpit` package's four are a good base) and change the least that fits.

## 2. Package or local
- **Local** (the user's own package, `~/.cockpit/packages/local/workflows/<id>.yaml`): for one project's needs. May name agents directly (`agent: Grunt`) and project commands. Skills are referenced with their package (`skill: cockpit/implement`).
- **A package** (a repo others install): must stay generic. Name `role:`s declared in its `cockpit.yaml` (with a suggested model each), never agents, never one project's commands, paths or machine.
If in doubt, it's local.

## 3. Write it
Follow `format.md`. Keep the step keys short and the `name:`s plain ("Write spec"). Put a one-line `#` comment above any `run:` step saying what it does. Every loop back from an agent step or group carries `max:`. Give each agent step a `prompt:` that names the files it reads and writes with `{{run.folder}}`.

## 4. Check it
- Walk it through once in prose with the user: step by step, what each writes, where each outcome goes, where it stops for them.
- Check every variable against the closed list in `format.md`; a `{{<step>.file}}` must name a step that writes a file before it.
- Quote any plain YAML value that contains `: ` (a colon and a space) or starts with `{` or `[`; a prompt like `Tests: focused only` breaks the file otherwise.
- The app validates on load and in the Workflows view (it names the first error and its line). Have the user open it there, or paste it into **Edit as YAML**.

## 5. Deliver
- **Local:** hand the YAML to the user to paste into the Workflows view (New workflow, then Edit as YAML), which saves and commits it in their local package. Write the file into `~/.cockpit/packages/local/workflows/` yourself only when the user says so and Cockpit is closed, since the app keeps that folder as a git repo and commits its own saves.
- **A package:** add the file under the package's `workflows/`, any new skills under `skills/`, the roles in `cockpit.yaml`, and describe it in the package's README and contract docs. Commit; push only when the user says so.
