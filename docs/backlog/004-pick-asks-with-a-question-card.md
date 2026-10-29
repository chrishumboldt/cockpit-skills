---
status: done
kind: ux
area: skills
added: 2026-10-29
done: 2026-10-30
---

# 004 · Pick items asks with a question card

In Run 2 (6b15e2) the Pick items step listed the ready items as plain text and waited for a typed answer, where the user expected a picker card. `skills/backlog-pick/SKILL.md` never mentions the session's question card, while `skills/preflight/SKILL.md` already says to use it (`AskUserQuestion`) whenever the answer is a pick between options.

## What to build
- **Picking items:** with 4 or fewer ready items, Pick items asks with a multi-select question card, one option per ready item. Each option's label is the number and title, and its description gives the kind, area and any `blockedBy` / `needs` ("needs 014 first"). The line about `idea` items waiting for shaping stays in the question text. With more than 4 ready items (a card holds at most 4 options), the step keeps the numbered list and the typed answer (numbers, a range or a filter).
- **The step 2 checks:** adding unpicked blockers, confirming a `needs` is met, and building a thin item anyway are each asked with a question card (yes / no, plus the card's own Other field).
- The skill's wording follows preflight's: use the card when the answer is a pick between options, and plain text when it needs the user's words.

## Acceptance criteria
- [ ] With 1–4 ready items, Pick items shows a multi-select card listing each one, and the picked options become the run's tickets.
- [ ] With 5 or more ready items, it shows the numbered list and accepts numbers, ranges and filters as before.
- [ ] A pick with an undone, unpicked blocker asks with a card whether to add it, and a pick with `needs` asks with a card whether it's met.
- [ ] An answer typed in the card's Other field ("all ux", "1-3") is handled like a typed answer.
- [ ] The skill's frontmatter still parses.

## Verified by
A Backlog 2 Shipped run in a scratch project with three ready items, one blocked by another: the card lists all three, picking the blocked one alone brings up a card asking to add its blocker, and the tickets match the answers. Then the same run with six ready items shows the numbered list.
