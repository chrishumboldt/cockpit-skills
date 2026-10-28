---
name: write-spec
description: The Write spec step of a Cockpit run. Turns the settled idea from the preflight notes, its transcript and design.md into spec.md in the run folder, with no questions asked.
disable-model-invocation: true
---

# Write spec

You run the Write spec step of a Cockpit run. Turn what the preflight settled into one spec file. You interview nobody: everything you need is in the files and the repo. The spec goes only to the path the step's prompt gives (`<run folder>/spec.md`); the run's gate shows it to the user next.

## 1. Read

- `preflight-notes.md` in the run folder: the idea, its decisions, terms and open questions. This is your primary source; a decision recorded there is settled.
- The preflight's transcript (the opening message names it) for detail the notes compress.
- `transcript-design.md` for the idea as first told, and `design.md`: when it holds a design, the spec matches its screens, states and copy.
- The repo: the glossary, the ADRs in the area, and the code the change touches, so the spec uses the project's terms and fits what exists.

## 2. Pick the test seams

A **seam** is the public boundary where a test observes behaviour without reaching inside. Choose where this change is tested: prefer seams the repo already tests at, as high as they go, and as few as will cover it (one is ideal). Name any new seam and why it is needed.

## 3. Write `spec.md`

```markdown
# <Title>

## Goal
The problem and who has it, then what changes for them when this ships. A short paragraph each.

## Non-goals
What this deliberately leaves out, each with a word on why.

## Behaviour
What the software does, as numbered statements a tester could check: "When a run's step fails, the run pauses at that step and its block reads 'Paused at step 3'." Group them under sub-headings by area when there are many. Cover the empty, error and edge cases, not only the happy path.

## Decisions
The implementation choices made: modules built or changed and their interfaces, data shapes, contracts between parts, and the reason for each where it isn't obvious. Name modules by their role, not by file path; inline a snippet only where it pins a decision (a type, a schema) more exactly than prose.

## Testing
The seams chosen in step 2, what each one proves, and the existing tests to follow as prior art.

## Acceptance
- [ ] Checkable criteria for the whole change: each one true or false by looking at the running software or a test.

## Open questions
What the preflight left open, with the recommendation for each. "None" when nothing is.
```

Use the glossary's terms throughout. Every decision in the preflight notes appears in the spec or is named in Non-goals; nothing is added that the notes and transcript don't support. Where they are silent and the spec needs an answer, pick the reading that fits the rest of the idea and list it under Open questions as an assumption.

## 4. When the gate sends it back

When the user requests changes, the message lists their notes (also kept in `gate-spec.md` in the run folder). Revise `spec.md` in place to address every note, and change nothing they didn't ask about.

## 5. Report

Call `submit_artifact` on the `cockpit` MCP server with kind `spec` and the spec's path, then `report_outcome` (`mcp__cockpit__report_outcome`) with a one-line summary ("Spec for run retries: 14 behaviours, 2 seams, 1 open question").
