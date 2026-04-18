---
name: academic-essay-style-check
description: Use when a draft.md exists and needs a style audit and fact-check before export. Scans for em dashes (banned), explanatory colons (banned), banned phrases and overused transitions, buzzword clichés, restricted-word overuse (corporate jargon cluster and vague-qualifier cluster capped at two uses per essay), and empirical-claim fact-check (verifies specific numbers, dates, people, institutions, policies, and rulings against authoritative sources via WebSearch). Preserves the banned-phrase, restricted-word, and fact-check rules verbatim from the original academic-essay-framework skill. Trigger when the user asks to check style, audit jargon, fact-check the draft, or polish before export. Produces style-audit.md with findings and suggested specific replacements.
---

# academic-essay-style-check

## What this skill does

This is the final prose audit in the academic essay workflow. It runs on an existing `draft.md` in the workspace and flags every style violation defined by the original `academic-essay-framework` skill: em dashes, banned phrases, overused transitions, buzzword clichés, and restricted-word cluster overuse. It does not rewrite structure or verify quotes. Framework structure is `academic-essay-framework-check`'s job, quote fidelity is `academic-essay-quote-verify`'s job, and new prose generation is `academic-essay-draft`'s job. This skill reads `draft.md`, produces `style-audit.md` with concrete findings and replacement suggestions, and stops there unless the user explicitly approves a rewrite pass.

## Inputs

- `draft.md` in the working directory. Required. If it is missing, tell the user to run `academic-essay-draft` first and stop.
- `topic.md` and `sources.md` if present. Used only for context when suggesting replacements, for example pulling a concrete date or proper noun from the primary source into a sentence that currently leans on a banned buzzword.

## Scan algorithm

Run these scans in order. Use `Grep` for literal-string matches against `draft.md` and `Bash` counts for cluster totals. Record every match with its line number for the audit file.

1. **Em dashes.** `Grep` for the literal `—` character (U+2014). Every hit is a finding. Also scan for the ASCII double-hyphen `--` in case the draft slipped one in and auto-format did not convert it. Skip matches inside fenced code blocks (those appear occasionally in footnotes or sidebar prose).
2. **Explanatory colons.** `Grep` for `: ` (colon followed by space) against `draft.md`. Each hit needs a qualitative judgment: is the colon introducing an idea, explanation, or continuation of the sentence (prohibited), or is it mechanical (allowed — times like `10:30`, ratios like `2:1`, chapter-subtitle form like `Title: Subtitle`, dialogue attribution)? Flag every non-mechanical hit in the audit as a violation with a suggested rewrite using a period, semicolon, or sentence restructure. Enumerative list colons in the middle of a sentence (e.g., `four dimensions: temporal, social, geographical, and uncertain`) are borderline; note them as "review — prefer restructure if the list is long or could stand as its own sentence, otherwise acceptable." Also scan for `URL:` and protocol schemes like `https:` as obvious mechanical uses that can be skipped.
3. **Banned phrases.** For each phrase in the "Banned phrases" subsection below, `Grep` case-insensitively against `draft.md`. Record the line, the matched phrase, and the surrounding sentence.
4. **Restricted-word clusters.** For each word in the corporate-jargon cluster and each word in the vague-qualifiers cluster, count occurrences across the whole draft. Use a word-boundary regex (e.g., `\bleverage\b`) so that substrings inside larger words do not inflate the count. Cluster totals beyond two are over budget. Report both the per-word counts and the cluster total.
5. **Overused transitions.** Count occurrences of `Moreover`, `Furthermore`, and `Additionally`. Flag any use beyond once per 800 words of draft body. Compute draft body word count with `wc -w draft.md`. Cap per transition word = max(1, floor(word_count / 800)). Flag any two consecutive paragraphs that both start with `However` or `Therefore`. Detect two consecutive paragraphs starting with the same transition using a multiline regex: `rg --multiline -P '(?m)^(However|Therefore)[^\n]*\n\n(?:^(?!#)[^\n]*\n+)*?(However|Therefore)'` — if the capture groups match the same word, flag. Flag any `X isn't the problem, Y is` construction and any rule-of-three list packed into a single sentence. For `X isn't the problem, Y is`: regex `\b(isn'?t|is not) (the|a) \w+,\s+\w+ is\b`. For rule-of-three-in-one-sentence: this is hard to detect mechanically; read the draft once end-to-end after mechanical scans and flag any sentence containing three comma-separated items in parallel form that appear rhetorical rather than enumerative.
6. **Fact-check empirical claims.** Read the draft end-to-end and identify every sentence that contains a checkable piece of fact: a specific number or statistic, a date, a named person, a named institution, a named policy or bill, a court ruling, a concrete historical event. For each, form a focused `WebSearch` query (e.g., `"EPA endangerment finding rescission 2026 date"` or `"Italy 33 hours climate education mandate year"`) and run it. Compare the top results to the claim and mark it with one of four statuses: **Verified** (authoritative sources report the claim directly), **Partially verified** (general framing supported but a specific detail looks off — note the detail), **Not found** (claim cannot be located in accessible sources — flag for the user to supply a primary source or drop the claim), or **Contradicted** (authoritative sources disagree — claim must be removed or rewritten). Prefer primary/authoritative sources (government agency pages, peer-reviewed journals, legal-scholarly databases, major news of record, institutional NGO publications). Treat advocacy blogs, opinion pieces, and paraphrased summaries as non-authoritative; a claim that only appears in those is not verified. Skip pure argumentative or interpretive sentences — this scan is for claims of fact, not claims of judgment. When an authoritative URL supports the claim, record the URL alongside the status.

Every finding cites the `draft.md` line number and proposes a concrete replacement grounded in the primary source or the essay's own vocabulary, not a generic synonym.

## The lifted rules

The five subsections below are lifted verbatim from `desktop/academic-essay-framework/SKILL.md`. They are the contract this skill enforces. Do not paraphrase, abridge, or reorder the lists.

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

### Fact-check for empirical claims

Every empirical claim in the draft has to be independently verifiable. Identify any claim that names a specific number, date, named person, institution, policy, court ruling, statistic, or concrete event, and confirm it against an authoritative source before treating it as settled. Prefer primary and authoritative sources in this order: government agency pages and official press releases, peer-reviewed journal articles, legal-scholarly databases (Sabin Center, Columbia Law Blog), news coverage of record (Reuters, Associated Press, major newspapers), and institutional NGO publications (EDF, UCS, NAAEE). Treat a claim as verified only when an authoritative source states it directly; treat a claim that only appears in advocacy blogs, opinion pieces, or paraphrased summaries as unverified and flag it.

Mark each checked claim with one of four statuses. Verified means the claim matches what authoritative sources report. Partially verified means the general framing is supported but a specific detail (number, date, scope) may be off. Not found means the claim cannot be located in accessible sources, in which case the student has to produce a primary source or drop the claim before submission. Contradicted means authoritative sources directly disagree with the claim, in which case the claim must be removed or rewritten.

This is especially load-bearing when the essay's argument rests on a specific empirical detail. A recent session caught a claim about an Italian Ministry of Education 2022 assessment finding no shift in student policy positions; the 33-hour climate-education mandate itself is real and verifiable, but the specific 2022 assessment finding could not be located in accessible sources and had to be flagged. Catching an unsupported claim before submission is cheaper than defending it after a reader checks it.

Fact-check is for claims of fact, not claims of judgment or interpretation. Skip pure argumentative sentences; apply the check to the specific, checkable pieces the argument rests on.

## Output file: `style-audit.md`

When the scan is done, write `style-audit.md` in the workspace. Plain markdown, no YAML frontmatter. One H1, three H2 sections in the order below. Each finding cites the `draft.md` line number and suggests a concrete replacement drawn from the essay's own sources or proper nouns wherever possible.

The exact file contents (strip the fence markers when writing).

```markdown
# Style audit

## Em dashes found
- `draft.md` line N: "<the sentence containing the em dash>"
  - Suggested rewrite: "<same sentence with the em dash replaced by a comma, period, or semicolon>"
- (repeat per finding, or write "None" if clean)

## Explanatory colons found
- `draft.md` line N: "<the sentence containing the explanatory colon>"
  - Suggested rewrite: "<same sentence with the colon replaced by a period, semicolon, or restructure>"
- `draft.md` line N: "<borderline enumerative colon sentence>"
  - Review only: mechanical enumeration; restructure preferred but not required.
- (repeat per finding, or write "None" if clean)

## Banned phrases found
- `draft.md` line N: matched phrase "<phrase>" in sentence "<the sentence>"
  - Suggested rewrite: "<concrete replacement that removes the banned phrase without weakening the claim>"
- (repeat per finding, or write "None" if clean)

## Restricted words over the cap
- Corporate jargon cluster total: X of 2 allowed
  - `leverage` × N (lines N, N, N)
  - `optimize` × N (lines N, N)
  - (only list words that actually appear)
  - Over-budget words to replace: <word> on lines N, N
    - Suggested replacement on line N: "<specific, historically grounded alternative>"
- Vague qualifiers cluster total: X of 2 allowed
  - (same shape)
- (write "Both clusters within cap" if clean)

## Facts checked
- `draft.md` line N: "<the claim sentence>"
  - Status: Verified / Partially verified / Not found / Contradicted
  - Source: <authoritative URL, or "none found">
  - Note: <one-line explanation of the finding; cite the specific number/date/name that matches or conflicts>
- (repeat per claim; write "No empirical claims flagged — argumentative prose only" if the draft contains no checkable facts)
```

Keep every suggested replacement specific. Do not suggest "use a more concrete word" without naming the word. When a banned phrase appears in a thesis or topic sentence, propose a replacement that preserves the claim's intellectual work, not just its grammar.

## Rewrite offer

After `style-audit.md` is written, tell the user the audit is saved and ask whether they want a rewrite pass that applies the suggested replacements back into `draft.md`. Do not rewrite silently.

If the user approves a rewrite:
1. Copy the current `draft.md` to `.archive/draft-<YYYYMMDDTHHMMSSZ UTC>.md` in the workspace before editing. This is non-negotiable. The user must be able to recover the pre-rewrite prose.
2. Apply the replacements from `style-audit.md` in place to `draft.md`.
3. Re-run the scan once to confirm zero em dashes, zero banned phrases, and both clusters back within cap. If anything is still over, report the residual findings and stop rather than cascade-rewriting.

If the user declines, leave `draft.md` untouched and stop.

## Termination condition

Stop when both are true:

- `style-audit.md` is written in the workspace.
- The user has either declined a rewrite, or approved one and the post-rewrite scan confirms zero em dashes, zero explanatory colons (non-mechanical), zero banned phrases, both clusters back within the two-uses cap, and zero fact-check claims in `Not found` or `Contradicted` status (user has resolved each flagged claim by supplying a source, rewriting, or dropping it).

Do not run framework-structure checks, quote verification, or export from this skill. Those are `academic-essay-framework-check`, `academic-essay-quote-verify`, and `academic-essay-export` respectively.
