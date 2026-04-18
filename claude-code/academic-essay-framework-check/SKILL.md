---
name: academic-essay-framework-check
description: Use when the user shares an outline or a draft and wants it checked against the 4-paragraph academic-essay framework (intro with hook/bridge/thesis, two body paragraphs each with argument/evidence/analysis, conclusion with restate/so-what). Produces a framework-vs-outline review table flagging specific gaps, historical inversions, or overbroad arguments. Trigger when the user asks to check an outline, run the framework check, or audit essay structure. Produces framework-review.md.
---

# academic-essay-framework-check

## What this skill does

This is the structural-check stage of the academic essay workflow. It takes a user-provided outline (or a just-written draft that needs a re-check) and maps it against the 4-paragraph academic-essay framework, producing a review table that flags specific gaps row by row before any drafting or polishing happens. This skill is not a drafting skill and not a style pass — it checks structure. Drafting lives in `academic-essay-draft` and line-level style rules live in `academic-essay-style-check`. Run the framework check first, substantive feedback second. Output is `framework-review.md` in the workspace.

## The 4-paragraph framework

Every essay uses four paragraphs: intro, body 1, body 2, conclusion. Each paragraph has a specific internal structure. Check every outline against this template before writing analysis or giving substantive feedback.

### Intro paragraph

Three moves, in order.

**1. Hook.** A concrete, grounded opening. Prefer a specific date, number, proper noun, or image drawn from the primary source. Avoid abstract framing sentences and do not open with "In today's world" style generalities. The strongest hooks anchor a single scene: a specific year, a survey data point, a dollar figure, an architectural detail, a named document.

**2. Bridge.** Tie the hook to the problem the essay addresses. One to three sentences. This is where historical context, the contradiction being exposed, or the stakes get stated plainly.

**3. Thesis.** A single claim that does real intellectual work. It must be an argument, not a description. The thesis should name a specific mechanism, not just announce a topic. Push back when a thesis repeats itself across multiple sentences, buries its strongest claim in the middle, or is historically inverted (claims cause-and-effect that the chronology does not support).

### Body paragraph

Same structure for both body paragraphs, built out of repeating argument-evidence-analysis units.

**1. Central argument (topic sentence).** One sentence stating the paragraph's claim. It must support the thesis without restating it. Each body paragraph should carry a distinct argumentative move.

**2. Evidence: one or two quotes.** Up to two direct quotes per body paragraph, each with its page number. Every quote must be verified word-for-word against the original text before analysis is built around it. If a quote in the outline does not match the source, flag it before proceeding.

**3. Analysis.** Each quote drives its own analysis, not the other way around. Connect the quote to the central argument first, then back up to the thesis. Do not use quotes as decoration placed after the argument is already made.

When the central argument requires more than one source to support it, the paragraph uses the argument-evidence-analysis flow twice, once for each source. The structure looks like this: central argument, then quote from source A with analysis connecting it to the argument and thesis, then quote from source B with analysis connecting it to the argument and thesis, then a closing sentence that ties both pieces of evidence together and returns to the thesis. The two quotes should do different work inside the argument. If both quotes make the same point, consolidate them and keep only the stronger one. If a paragraph needs three or more quotes to support a single argument, the argument is probably too broad and should be split across both body paragraphs.

### Conclusion

Two moves.

**1. Restate the thesis in new language.** Do not copy the intro's thesis sentence. Rephrase the argument using the terms earned during the body paragraphs.

**2. So what.** What does the argument mean for the world right now? Connect the essay's specific claim to a present-day mechanism, policy, or pattern. This is the stakes, not a summary of the essay.

## Outline review workflow

When the user shares an outline, run the framework check first and substantive feedback second. The useful format is a small table mapping each required move to what the outline actually contains, marking each row with a check mark or a specific gap.

The exact review-table template (strip the fence markers when writing into `framework-review.md`).

```markdown
| Framework | Outline | Status |
|---|---|---|
| Intro: hook → bridge → thesis | (describe what the outline has) | ✅ or specific gap |
| Body ¶1: argument → evidence (one or two quotes) → analysis tying to thesis | (describe) | ✅ or gap |
| Body ¶2: argument → evidence (one or two quotes) → analysis tying to thesis | (describe) | ✅ or gap |
| Conclusion: restate thesis → so what | (describe) | ✅ or gap |
```

After the table, move to substantive feedback in prose: thesis grounding, evidence gaps, sequencing between paragraphs, and specific line-level fixes.

## Common gaps to watch for

Flag these patterns explicitly in the table's Status column or in the prose feedback that follows:

- Only one body paragraph written — the framework requires two with distinct argumentative moves.
- Three or more quotes crammed into a single body paragraph — a signal that the argument is too broad and should be split across both body paragraphs.
- Two quotes in a paragraph doing the same work rather than different work — consolidate and keep the stronger one.
- Thesis that is descriptive rather than argumentative — announces a topic instead of naming a mechanism.
- Hook that is abstract rather than grounded in concrete detail — no date, number, proper noun, or image from the primary source.
- Conclusion that summarizes instead of stating "so what" — no connection to a present-day mechanism, policy, or pattern.
- Historical inversion in the thesis where the claimed cause came after its supposed effect — the chronology does not support the cause-and-effect.
- Scholar whose framework does not match the source mechanics — a recurring professor critique that should drive a scholar swap. When this gap fires, flag it in the Status column and recommend re-running `academic-essay-scholar-match`. Do not swap scholars inside this skill.

## Output file: `framework-review.md`

Write `framework-review.md` in the workspace with the filled-out review table at the top and prose feedback beneath it. The exact file structure (strip the fence markers when writing).

```markdown
# Framework review

## Review table

| Framework | Outline | Status |
|---|---|---|
| Intro: hook → bridge → thesis | (what the outline has) | ✅ or specific gap |
| Body ¶1: argument → evidence (one or two quotes) → analysis tying to thesis | (what the outline has) | ✅ or gap |
| Body ¶2: argument → evidence (one or two quotes) → analysis tying to thesis | (what the outline has) | ✅ or gap |
| Conclusion: restate thesis → so what | (what the outline has) | ✅ or gap |

## Substantive feedback

Prose notes on thesis grounding, evidence gaps, sequencing between paragraphs, and specific line-level fixes. Name the page number, the sentence, or the word when suggesting a fix.
```

Fill in every row of the review table. Do not leave a row blank, and do not collapse rows into a summary sentence. If a paragraph is missing from the outline, record the Outline cell as "missing" and flag the gap in the Status column.

## Termination condition

Stop and hand off once both are true:

- The review table and prose feedback are written to `framework-review.md`.
- The user has either acknowledged the review (some version of "got it" / "understood" / "I'll fix it") or explicitly asked to move on to drafting.

Once `framework-review.md` is written and the user acknowledges, tell the user the file is saved and that the next skill in the workflow (`academic-essay-draft`) will take the revised outline from here. If the user asks you to re-check a subsequently revised outline, run the workflow again and overwrite `framework-review.md`. Do not begin drafting, polishing style, or verifying quotes — those are downstream skills' jobs.
