---
name: academic-essay-scholar-match
description: Use when evaluating whether a scholar's framework tightly matches the mechanics of the primary source an essay analyzes. Preserves the scholar-source matching logic from the original academic-essay-framework skill (Winner on design encoding politics, Roy on sovereign planning power, Bullard and Pulido on environmental burdens by race and class). Trigger when the user has a candidate scholar and a primary source and is deciding whether the pairing holds, or when multiple candidates compete. Produces scholars.md with the chosen scholar, justification, and rejected candidates with reasons.
---

# academic-essay-scholar-match

## What this skill does

This is the fourth stage of the academic essay workflow. It runs after the user already has one or more candidate scholars in hand — either surfaced by `academic-essay-find-sources`, named by the professor, or drawn from a course reading list. It does **not** search for new scholars. It evaluates whether a candidate scholar's framework tightly matches the mechanics of the primary source the essay analyzes, chooses between competing candidates, and records the decision in `scholars.md`. When the chosen scholar does not fit, this skill names the mismatch directly and recommends a swap rather than polishing a weak pairing.

## Scholar-source mechanics matching

For source-analysis essays, scholars must align tightly with the specific mechanics of the primary source. This is a recurring professor critique and a frequent cause of weak essays. If a scholar's framework requires elements the source does not supply, the scholar is wrong for the essay and should be swapped.

Examples of what this means in practice:
- A framework that requires a persistent community accumulating harm over time (slow violence) cannot be applied cleanly when the displacement analyzed in the source preceded the infrastructure failure. The community whose harm the framework describes is not present in the source.
- A framework about design encoding political hierarchy (Winner, "Do Artifacts Have Politics?") fits tightly when the source is an architectural rendering, drawing, feasibility study, or design criteria document.
- A framework about sovereign planning power and which knowledge enters the planning record (Roy on the state of exception) fits tightly when the source is a commissioned study whose findings were generated but never acted upon.
- A framework about the politics of race and place (Bullard, Pulido) fits tightly when the source reveals uneven distribution of environmental burdens or benefits across racial or class lines.

When evaluating scholar choices, ask: does this scholar's framework name the exact mechanism the source reveals? If not, suggest a better-matched scholar and explain the mismatch directly.

## The key diagnostic question

Every evaluation in this skill reduces to one question: **does this scholar's framework name the exact mechanism the source reveals?** If the framework requires a persistent community and the source has none, the answer is no. If the framework is about design encoding politics and the source is a design document, the answer is yes. Do not soften a "no" into a "maybe" to keep a scholar the user has grown attached to — a mismatched scholar surfaces in professor feedback almost every time, and it is cheaper to swap now than to rewrite the essay later.

When multiple candidates compete, answer the diagnostic question for each one and pick the scholar whose framework names the tightest mechanism. Prefer to find the tighter fit; if you cannot, pause rather than force a pick. If two candidates genuinely tie, surface the tie to the user rather than picking silently.

## Autoplan contract

When invoked in autoplan mode (`auto=true`), two conditions pause and return control to the orchestrator:
- (a) Two or more candidate scholars genuinely tie on the diagnostic question.
- (b) No candidate passes the diagnostic against the primary source's mechanics.

In both cases, write what you have so far to `scholars.md` (including rejected candidates and reasons), and surface the tie or the floor-miss explicitly in the output so the orchestrator can route the taste-gate prompt to the user.

## Output file: `scholars.md`

When the evaluation is done, write `scholars.md` in the working directory with the `# Scholars` H1 and exactly these three H2 sections, in this order. The exact file contents (strip the fence markers when writing).

```markdown
# Scholars

## Chosen scholars
- **Scholar name** — *Key work title* — Framework in one sentence — Why this fits the primary source's mechanics.

## Rejected candidates
- **Scholar name** — *Key work title* — Framework in one sentence — Why this does not fit (name the missing mechanism).

## Why the pairing holds
One paragraph connecting the chosen scholar's framework to the specific mechanic the primary source reveals. Name the source by title and the framework by its operative concept.
```

Each scholar entry is a single bullet in the form `- **Name** — *Key work* — Framework — Fit or mismatch reason`. Use em dashes between the four fields inside each bullet (this is the scholars.md output only; em dashes are banned from the finished essay prose). If a section has no entries, write `None` as the entire body of that section instead of a bullet list.

## Termination condition

Stop evaluating and write `scholars.md` as soon as either is true:

- The user confirms a choice ("go with Winner," "drop Nixon, keep Roy," "that's the pairing").
- The orchestrator has enough data to proceed: at least one scholar has passed the diagnostic question against the primary source, and any rejected candidates have a named mismatch reason recorded.

After writing, tell the user the file is saved and that the next skill in the workflow (`academic-essay-framework-check`) will take it from here. Do not begin drafting, searching for more scholars, or verifying quotes — those are downstream skills' jobs.
