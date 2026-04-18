---
name: academic-essay-draft
description: Use when framework-review.md and quote-verification.md are clean (or their issues acknowledged) and the user is ready to write the full essay draft. Writes the essay following the voice rules from the original academic-essay-framework skill (historical, grounded, argumentative, varied sentence rhythm), using only verified quotes. Trigger when the user asks to draft, write, or build the essay after outline and evidence are settled. Produces draft.md.
---

# academic-essay-draft

## What this skill does

This is the seventh stage of the academic essay workflow. Upstream stages have already produced `topic.md`, `materials.md`, `scholars.md`, a quote-ready outline, a clean `framework-review.md`, and a clean `quote-verification.md`. With the argument scaffolded and every quote verified word-for-word, this skill writes the full essay as prose. It lifts the voice rules from the original `academic-essay-framework` skill and applies them sentence by sentence. It does **not** re-check the framework, does **not** re-verify quotes, and does **not** audit for em dashes or banned phrases. Those responsibilities belong to upstream and downstream skills. The output is a single `draft.md` in the working directory: the full essay in free-form prose, ready for `academic-essay-style-check` to audit.

## Preconditions

Before drafting, confirm the following inputs exist in the working directory. If any are missing, stop and tell the user which upstream stage needs to run first.

- `topic.md` — the clarified topic, assignment prompt, and angle.
- `scholars.md` — the matched scholar or scholars, with the framework each one supplies.
- An outline, if the user produced one. The draft should follow it paragraph for paragraph if present; if absent, structure the draft using the intro → body 1 → body 2 → conclusion framework directly.
- `framework-review.md` — must be clean (every row marked ✅) or the user must have explicitly acknowledged the remaining gaps and asked to draft anyway.
- `quote-verification.md` — must be clean (every quote verified at its cited page) or the user must have explicitly acknowledged the remaining issues and asked to draft anyway.

`materials.md` is optional at this stage; by now the content has been digested into `topic.md`, `scholars.md`, and the verified quotes.

## Voice rules

The prose should be precise, historical, and grounded. Direct and concrete. Dates, numbers, proper nouns, specific places. Argumentative rather than descriptive, with every sentence advancing a claim. No throat-clearing sentences that delay the point. Sentence length should vary, mixing short punchy claims with longer analytical sentences.

These rules apply to every paragraph of the draft. Intro paragraphs anchor the hook in a specific scene: a year, a survey figure, a dollar amount, a named document, a line from a design brief. Body paragraphs lead with the argument, then deploy the verified quote, then connect the quote back to the argument and up to the thesis in the same breath. Conclusions restate the thesis in new language earned from the body and land a concrete so-what tied to a present-day mechanism, policy, or pattern. Do not summarize; do not pad.

The style-check skill (`academic-essay-style-check`) handles em dashes, banned phrases, and restricted-word budgets. Write naturally in this skill; the downstream audit will catch violations. Do not try to pre-audit the prose against those lists inside this skill.

## Use only verified quotes

Every direct quote in `draft.md` must appear in `quote-verification.md` with a ✅ status and a confirmed page number. Do not introduce a new quote mid-draft. Do not paraphrase a source so tightly that the paraphrase is functionally a direct quote without quotation marks.

### Loop-back rule

If, while drafting, you realize the argument needs a new quote that is not in `quote-verification.md`, **stop drafting that paragraph**. Signal the orchestrator that `academic-essay-quote-verify` must re-run against the new candidate quote. Record the candidate quote, the source, and the suspected page in a `# Pending quotes` section at the bottom of the partial draft, then return control. Do not introduce the quote into the prose until verification is complete and `quote-verification.md` has been updated. Inventing or approximating a quote mid-draft is the single fastest way to poison the essay; the loop-back is the correct move even if it interrupts flow.

## Drafting order

Write the essay in framework order: intro, body 1, body 2, conclusion. Do not start with the conclusion and work backward; the thesis earned during intro-writing is what the conclusion restates in new language, and writing them out of order tends to produce a conclusion that echoes the intro rather than advancing beyond it.

If the user asks for a single paragraph rather than the full draft ("write the first body paragraph using the verified quotes"), produce that paragraph in isolation and append it to `draft.md` at the correct slot, leaving other paragraphs as placeholders the user can fill in on a later pass.

## Output file: `draft.md`

When drafting is complete (or when the user stops the session), write `draft.md` in the working directory. The file is the full essay as markdown prose. No fenced schema, no per-section headers, no meta-commentary inside the file. The first line is the essay title if the user has named one; otherwise start with the intro paragraph directly. Paragraphs are separated by blank lines. Quotes stay in double quotation marks with parenthetical page citations in the style the course uses (most commonly `(Author page)` or `(Source page)`).

If the loop-back rule fired and the draft is partial, append a `# Pending quotes` section at the very bottom listing each candidate quote that needs verification before the draft can be completed. Remove this section on the next pass once verification clears.

## Termination condition

Stop and write `draft.md` when one of the following happens: all four paragraphs are complete, or the user says some version of "that's enough for now" / "save what you have" / "stop" / "pause", or the loop-back rule fires and verification must run before drafting can continue. After writing, tell the user the file is saved, summarize which paragraphs are complete and which (if any) are pending, and hand off to `academic-essay-style-check` for the em-dash, banned-phrase, and restricted-word audit. Do not run the style check inside this skill, and do not attempt export to `.docx` — that is `academic-essay-export`'s job.
