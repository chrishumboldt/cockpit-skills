---
name: project-setup
description: "Gets a repo ready for Cockpit's workflows, once, from the chat: finds its type-check and focused-test commands, checks the base branch accepts a push, and proposes a Cockpit section for its agent docs. Use when asked to 'set up this repo for Cockpit', or before a project's first run."
disable-model-invocation: false
---

# Project setup

You get a repo ready for Cockpit's workflows. They need three things the repo has to tell them: how to check a change, where a finished run lands, and the repo's own terms. Find them, show the user what you found, and write it down only when they agree.

## 1. Find the commands
Read the agent docs (`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`), the manifest (`package.json`, `go.mod`, `pyproject.toml`, `Cargo.toml`, a `Makefile`) and the CI workflows. Work out:
- **Type check** (when the language has one): `tsc --noEmit`, `go vet ./...`, `mypy .`.
- **Focused tests**: how to run only the tests for some files or a package (`vitest related <files>`, `go test ./pkg/...`, `pytest tests/test_x.py`), not just the whole suite.
- **End-to-end tests**, when the repo has them, and how to run one spec.
- **Limits**: whether the machine needs worker caps; ask the user, since the repo can't say.

Run each command once on a small target to check it works. A command you couldn't run goes down as unchecked.

## 2. Check where runs land
A finished run merges into the branch it started from and pushes it. Check:
- there is an `origin` remote (`git remote get-url origin`);
- the branch accepts a direct push: on GitHub, `gh api repos/<owner>/<repo>/branches/<branch>/protection` answers 404 when it's unprotected. When it needs pull requests, tell the user to start runs from a working branch (`beta`, `develop`) instead.

## 3. Note the terms
Say whether the repo has a glossary (`GLOSSARY.md` or one its docs point to) and ADRs (`docs/adr/`). They are optional; the skills use them when they exist. Offer to start a `docs/backlog/` only when the user wants to add an item now.

## 4. Propose, then write
Show the user the section you would add to the agent docs the repo already uses (`CLAUDE.md` when it has none):

```markdown
## Checking a change
- Type check: `<command>`
- Focused tests: `<command>` (for <files | a package>)
- End-to-end: `<command>` (one spec: `<command>`)
- Limits: <worker caps>, or none
- Runs land on `<branch>`.
```

Write it when they agree, with their changes. Commit it on the current branch only when they ask.

## 5. Say what's left
One line each for what you couldn't check, and what the user still has to do (a protected branch, a missing test runner).
