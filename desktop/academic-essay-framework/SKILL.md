---
name: academic-essay-framework
description: Apply a rigorous 4-paragraph framework and strict writing-style rules for academic essays, especially source-analysis and argumentative papers in humanities and social-science courses (environmental justice, urban planning, history, geography, sociology). Use whenever the user asks for help with an essay outline, thesis, introduction, body paragraph, or conclusion; pastes a draft for feedback; asks to "check" or "ground" a thesis; asks to fit quotes into an argument; shares a humanities or social-science assignment prompt; or asks Claude to draft or revise academic prose. The framework specifies exact internal moves per paragraph, mandates word-for-word quote verification against source PDFs, requires tight scholar-source mechanics matching, bans em dashes in output, and enforces banned phrases and restricted corporate-jargon words. Trigger aggressively whenever the request resembles academic writing help, even if the framework is not named.
---

# Academic Essay Framework

A rigorous framework for academic essays, tuned especially for source-analysis and argumentative papers in humanities and social-science courses. Use it whenever helping with academic writing, from outline review through final draft editing.

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

## Scholar-source mechanics matching

For source-analysis essays, scholars must align tightly with the specific mechanics of the primary source. This is a recurring professor critique and a frequent cause of weak essays. If a scholar's framework requires elements the source does not supply, the scholar is wrong for the essay and should be swapped.

Examples of what this means in practice:
- A framework that requires a persistent community accumulating harm over time (slow violence) cannot be applied cleanly when the displacement analyzed in the source preceded the infrastructure failure. The community whose harm the framework describes is not present in the source.
- A framework about design encoding political hierarchy (Winner, "Do Artifacts Have Politics?") fits tightly when the source is an architectural rendering, drawing, feasibility study, or design criteria document.
- A framework about sovereign planning power and which knowledge enters the planning record (Roy on the state of exception) fits tightly when the source is a commissioned study whose findings were generated but never acted upon.
- A framework about the politics of race and place (Bullard, Pulido) fits tightly when the source reveals uneven distribution of environmental burdens or benefits across racial or class lines.

When evaluating scholar choices, ask: does this scholar's framework name the exact mechanism the source reveals? If not, suggest a better-matched scholar and explain the mismatch directly.

## Quote verification

Verify every quote word-for-word against the original PDF before giving feedback or writing analysis around it. Search the source, find the exact page, and confirm the quote matches. If it does not, flag the correct wording and the correct page number clearly.

This has caught real misattributions in past work. For example, a Winner quote presented as "embody specific forms of authority and domination" when the actual page 121 phrasing reads "embody specific forms of power and authority." Always check before building analysis. The habit also catches paraphrases that have been promoted to direct quotes, which weaken the essay when a reader checks the citation.

When the primary source is available as a PDF (uploaded to the conversation, attached to project knowledge, or present in the working directory), search it directly. Use whatever extraction path the environment provides: the built-in PDF reading in Claude Desktop, or `pdftotext` / Claude Code's native PDF Read support when running in Claude Code. Confirm the exact wording from the extracted text. When the source is an outside scholar, verify against the primary text rather than a secondary summary.

## Outline review workflow

When the user shares an outline, run the framework check first and substantive feedback second. The useful format is a small table mapping each required move to what the outline actually contains, marking each row with a check mark or a specific gap.

| Framework | Outline | Status |
|---|---|---|
| Intro: hook → bridge → thesis | (describe what the outline has) | ✅ or specific gap |
| Body ¶1: argument → evidence (one or two quotes) → analysis tying to thesis | (describe) | ✅ or gap |
| Body ¶2: argument → evidence (one or two quotes) → analysis tying to thesis | (describe) | ✅ or gap |
| Conclusion: restate thesis → so what | (describe) | ✅ or gap |

Common gaps to watch for: only one body paragraph written; three or more quotes crammed into a single body paragraph (a signal that the argument is too broad and should be split); two quotes in a paragraph doing the same work rather than different work; thesis that is descriptive rather than argumentative; hook that is abstract rather than grounded in concrete detail; conclusion that summarizes instead of stating "so what"; historical inversion in the thesis where the claimed cause came after its supposed effect; scholar whose framework does not match the source mechanics.

After the table, move to substantive feedback in prose: thesis grounding, evidence gaps, sequencing between paragraphs, and specific line-level fixes.

## Working style when helping with essays

Run the framework check first, then substantive feedback. When asked to ground a thesis, identify the strongest sentence carrying the argument and build around it rather than rewriting the whole paragraph. When asked to fit quotes into an outline, verify the quotes exist in the source PDFs at the cited pages before building analysis around them. Push back when a claim is weak, historically inverted, or imprecise; correct rather than polish. Prefer tables for outline-vs-framework checks and prose for substantive edits. Be specific about fixes by naming the page number, the sentence, or the word. When a scholar's framework does not fit the source mechanics, name the mismatch directly and suggest an alternative.

When the user has received professor feedback, take it seriously and integrate it into the revision plan. Professor feedback that a scholar's framework does not fit the source is nearly always correct and should drive a scholar swap rather than a defense of the original choice.

## Writing style for drafts and revisions

The prose should be precise, historical, and grounded. Direct and concrete. Dates, numbers, proper nouns, specific places. Argumentative rather than descriptive, with every sentence advancing a claim. No throat-clearing sentences that delay the point. Sentence length should vary, mixing short punchy claims with longer analytical sentences.

### Em dash ban

Do not use em dashes anywhere in academic writing output. Use commas, periods, or semicolons instead. This is a hard rule that applies to drafts, revisions, and any prose intended for the essay itself.

### Explanatory colon ban

Do not use a colon to introduce an idea, explanation, or continuation of the sentence. Examples to avoid: writing "The problem" followed by a colon and then "manufactured doubt," or writing "Two structural vulnerabilities made the rollback possible" followed by a colon and then "environmental harm felt distant, and climate science felt complex." Replace with a period and a new sentence, a semicolon, or a sentence restructure that makes the relationship explicit. Colons remain allowed only for mechanical uses where the colon carries no explanatory weight, such as times (10:30), ratios (2:1), chapter or subtitle form ("Title: Subtitle"), and direct dialogue attribution. Enumerative list colons in the middle of a sentence, for instance listing "four dimensions of distance" followed by their names, are borderline; prefer restructuring when possible, but do not treat them as hard violations when the list is short and flows grammatically.

### Banned phrases

Never use any of the following under any circumstance, in any phrasing, in any context. This applies to the essay output and also to meta-commentary Claude writes alongside the essay.

Meta-commentary and disclaimers to avoid: "It is important to note that," "This underscores the importance of," "It cannot be denied that," "As of my knowledge cutoff," "But here's the catch," "And the X (benefit, mistake, big lesson)?" as an emphasis-via-question move, "they don't just X, they start Y," "They don't need X, they need Y," "You not only X, you know you can Y," "Because the transformation isn't X. It's in the Y.," "A aren't X. They're Y.," and "That's why it's just an 'X', but it's a 'Y'."

Generic openings and closings to avoid: "In today's fast-paced world," "In this ever-evolving landscape," "In the digital age," "In conclusion," "To summarize," "Finally," "Let's delve into," "delve deeper," "At its core," and "at the core."

Overused transitions to avoid: "Moreover," "Furthermore," and "Additionally" more than once per 800 words; consecutive paragraphs starting with "However" or "Therefore"; "X isn't the problem, Y is" constructions; and rule-of-three lists packed into a single sentence.

Buzzword clichés to avoid: "ever-evolving landscape," "dynamic world of," "digital realm," "in the realm of," "uncharted waters," "embark on a journey," "treasure trove of information," and "game-changer" unless backed by specific metrics.

### Restricted words, maximum two uses per essay

The following words are allowed but should be rare. Using more than two from any cluster in the same essay is over budget, and the draft should be revised to replace the excess with concrete language.

Corporate jargon cluster: leverage, optimize, enhance, utilize, synergy, deliverables, holistic, capability, pivotal, crucial, groundbreaking, cutting-edge, explore, delve, ensure, foster, embark.

Vague qualifiers: significant, relevant, dynamic, innovative, comprehensive, robust, streamlined.

When editing drafts, flag any of these words that appear more than twice and suggest specific replacements with concrete, historically grounded language.

## Tools and primary-source handling

Read primary source and course reading PDFs before building analysis, using whatever PDF tooling the current environment provides. When the final output needs to be delivered as a Word document, use the docx skill at `/mnt/skills/public/docx/SKILL.md` when running in Claude Desktop, or `pandoc` / `python-docx` via the shell when running in Claude Code. The standard formatting expectation for humanities essays is double-spaced, page numbers, Times New Roman 12-point type.

When the essay is source-analysis of an archival document, verify not just the quotes but the dates and provenance. Catching a 1969 document versus a 1975 document can reshape the entire argument, as can catching which parcel, box, or folder a given document belongs to.
