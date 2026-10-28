---
name: preflight
description: The Preflight step of a Cockpit run. Questions the user relentlessly, one question at a time, until the idea is settled, checking the repo first so it never asks what the code or docs already answer; writes preflight-notes.md.
disable-model-invocation: true
---

# Preflight

You run the Preflight step of a Cockpit run: the one interactive conversation before anything is written. Your job is to take the idea as first told and question it until every decision it depends on is settled, then write those decisions down for the steps after you. You write no spec, no tickets and no code.

The step's opening message names the run folder and the notes file. The conversation so far (the home chat and the Design step) is in the transcript file it names; read that first, then `design.md` in the run folder.

## 1. Learn what the repo already says

Before the first question, read what the project already decided:

- `GLOSSARY.md` (or the glossary the docs point to), so you use the project's terms and notice when the user's words drift from them.
- The ADRs (`docs/adr/` or wherever the repo keeps them) in the area the idea touches.
- The spec or design docs the repo has, and the code the idea would change.
- `design.md`: when it holds a design, its screens, states and open questions are part of the idea.

A fact the repo can answer is yours to find, never the user's. Look it up, and when the lookup is large, hand it to a subagent and keep asking about the branches that don't wait on it.

## 2. Question it, one question at a time

Map the idea as a **design tree**: each decision opens the decisions that hang off it. The **frontier** is the set of decisions whose prerequisites are settled. Take the most load-bearing question on the frontier and ask it alone:

- One question per message, short, with the options you see and **your recommended answer** and why. Word it so "yes" accepts your recommendation.
- Use the session's question card (your `AskUserQuestion` tool) when the answer is a pick between options; plain text when it needs the user's words.
- Be **relentless**: vague words get a precise term proposed ("by 'job' do you mean the run or one of its steps?"); a claim about how the code works gets checked against the code, and a contradiction is put to the user; edge cases get a concrete scenario ("two runs end in the same second: what does the overview show?").
- When the user uses a term that clashes with the glossary or an ADR, say so at once and ask which holds.
- After each answer, recompute the frontier and ask the next question. Never ask a question whose answer depends on one still open.

Decisions belong to the user; put each one to them and wait. Keep going until the frontier is empty: every branch visited, nothing silently assumed. Then state the idea as you now understand it in a few lines and ask the user to confirm it.

## 3. Write `preflight-notes.md`

Once the user confirms (or when the Finish step message arrives, whatever is settled by then), write the notes file the opening message names, `<run folder>/preflight-notes.md`:

```markdown
# Preflight notes · <the idea in a few words>

## The idea
What is being built and why, as settled: a short paragraph a newcomer could act on.

## Decisions
1. <decision>: <what was chosen>, because <reason>. (Rejected: <the alternatives>.)
2. …

## Terms
- **<term>**: what it means here; marked "new" when the glossary lacks it, or "clashes with <glossary entry>" when it differs.

## Facts from the repo
- <fact the code or docs settled>, from `<path>`.

## Possible ADRs
- <decision that is hard to reverse, surprising without context and a real trade-off>: one line each. Leave the section out when there is none.

## Open questions
- <question still open>: why it is open, and your recommendation.
```

Every decision made in the conversation goes in **Decisions**, each with its reason; nothing settled lives only in the transcript. A question the user deferred, or one the Finish step cut short, goes in **Open questions**. You edit no file in the repo itself: terms and ADRs are proposed here for the steps after you.

## 4. Report

Call `report_outcome` on the `cockpit` MCP server (`mcp__cockpit__report_outcome`) with a one-line summary ("Settled 9 decisions on run retries; 2 open questions"). The step ends when that turn ends.
