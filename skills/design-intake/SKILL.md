---
name: design-intake
description: The Design step of a Cockpit run. Asks once whether the work comes from a design, studies the link or image given, and writes design.md for the later steps.
---

# Design intake

You open a Cockpit run. Find out whether the user is working from a design, and hand what it shows to the steps after you through one file. The step's opening message names the run folder.

1. **Ask one question**, word for word: "Are you working from a design? Paste a link or an image, or say no."
2. **On no** (any decline: "no", "none", "skip"): write `<run folder>/design.md` as the single line `No design: the run starts from the idea alone.`, call `cockpit:report_outcome` with outcome `NONE`, and stop. Ask nothing more.
3. **On a link or an image**, open it with what this session can reach: an image or a public URL always; a claude.ai artifact or a Figma file only through its connector or MCP server, when this session has one. When you can't open it, say so plainly and ask for a screenshot instead; treat a second decline as no.
4. **Study it** until you can name every screen in it, then write `<run folder>/design.md`:
   - **Source**: the link, or the image's file name or path.
   - **Screens**: each one, what it is for.
   - **Components**: the parts each screen uses, and what repeats across screens.
   - **States**: empty, loading, error, selected, and any other the design draws.
   - **Copy**: headings, labels and messages, quoted exactly.
   - **Open questions**: what the design leaves unclear or contradicts.
5. **Report**: call `cockpit:report_outcome` with outcome `DESIGN` and a one-line summary ("Three screens of an inbox, grouped by thread").

`design.md` is written on every path: the steps after you read it.
