---
name: academic-essay-style-check
description: Use when a draft.md exists and needs a style audit before export. Scans for em dashes (banned), banned phrases and overused transitions, buzzword clichés, and restricted-word overuse (corporate jargon cluster and vague-qualifier cluster capped at two uses per essay). Preserves the banned-phrase and restricted-word lists verbatim from the original academic-essay-framework skill. Trigger when the user asks to check style, audit jargon, or polish the draft. Produces style-audit.md with findings and suggested specific replacements.
---

# academic-essay-style-check

## What this skill does

This is the final prose audit in the academic essay workflow. It runs on an existing `draft.md` in the workspace and flags every style violation defined by the original `academic-essay-framework` skill: em dashes, banned phrases, overused transitions, buzzword clichés, and restricted-word cluster overuse. It does not rewrite structure or verify quotes. Framework structure is `academic-essay-framework-check`'s job, quote fidelity is `academic-essay-quote-verify`'s job, and new prose generation is `academic-essay-draft`'s job. This skill reads `draft.md`, produces `style-audit.md` with concrete findings and replacement suggestions, and stops there unless the user explicitly approves a rewrite pass.

## Inputs

- `draft.md` in the working directory. Required. If it is missing, tell the user to run `academic-essay-draft` first and stop.
- `topic.md` and `sources.md` if present. Used only for context when suggesting replacements, for example pulling a concrete date or proper noun from the primary source into a sentence that currently leans on a banned buzzword.

## Scan algorithm

Run these scans in order. Use `Grep` for literal-string matches against `draft.md` and `Bash` counts for cluster totals. Record every match with its line number for the audit file.

1. **Em dashes.** `Grep` for the literal `—` character (U+2014). Every hit is a finding. Also scan for the ASCII double-hyphen `--` in case the draft slipped one in and auto-format did not convert it.
2. **Banned phrases.** For each phrase in the "Banned phrases" subsection below, `Grep` case-insensitively against `draft.md`. Record the line, the matched phrase, and the surrounding sentence.
3. **Restricted-word clusters.** For each word in the corporate-jargon cluster and each word in the vague-qualifiers cluster, count occurrences across the whole draft. Use a word-boundary regex (e.g., `\bleverage\b`) so that substrings inside larger words do not inflate the count. Cluster totals beyond two are over budget. Report both the per-word counts and the cluster total.
4. **Overused transitions.** Count occurrences of `Moreover`, `Furthermore`, and `Additionally`. Flag any use beyond once per 800 words of draft body. Flag any two consecutive paragraphs that both start with `However` or `Therefore`. Flag any `X isn't the problem, Y is` construction and any rule-of-three list packed into a single sentence.

Every finding cites the `draft.md` line number and proposes a concrete replacement grounded in the primary source or the essay's own vocabulary, not a generic synonym.

## The lifted rules

The three subsections below are lifted verbatim from `desktop/academic-essay-framework/SKILL.md`. They are the contract this skill enforces. Do not paraphrase, abridge, or reorder the lists.

### Em dash ban

Do not use em dashes anywhere in academic writing output. Use commas, periods, or semicolons instead. This is a hard rule that applies to drafts, revisions, and any prose intended for the essay itself.

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

## Output file: `style-audit.md`

When the scan is done, write `style-audit.md` in the workspace. Plain markdown, no YAML frontmatter. One H1, three H2 sections in the order below. Each finding cites the `draft.md` line number and suggests a concrete replacement drawn from the essay's own sources or proper nouns wherever possible.

The exact file contents (strip the fence markers when writing).

```markdown
# Style audit

## Em dashes found
- `draft.md` line N: "<the sentence containing the em dash>"
  - Suggested rewrite: "<same sentence with the em dash replaced by a comma, period, or semicolon>"
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
```

Keep every suggested replacement specific. Do not suggest "use a more concrete word" without naming the word. When a banned phrase appears in a thesis or topic sentence, propose a replacement that preserves the claim's intellectual work, not just its grammar.

## Rewrite offer

After `style-audit.md` is written, tell the user the audit is saved and ask whether they want a rewrite pass that applies the suggested replacements back into `draft.md`. Do not rewrite silently.

If the user approves a rewrite:
1. Copy the current `draft.md` to `.archive/draft-<timestamp>.md` in the workspace before editing. This is non-negotiable. The user must be able to recover the pre-rewrite prose.
2. Apply the replacements from `style-audit.md` in place to `draft.md`.
3. Re-run the scan once to confirm zero em dashes, zero banned phrases, and both clusters back within cap. If anything is still over, report the residual findings and stop rather than cascade-rewriting.

If the user declines, leave `draft.md` untouched and stop.

## Termination condition

Stop when both are true:

- `style-audit.md` is written in the workspace.
- The user has either declined a rewrite, or approved one and the post-rewrite scan confirms zero em dashes, zero banned phrases, and both clusters back within the two-uses cap.

Do not run framework-structure checks, quote verification, or export from this skill. Those are `academic-essay-framework-check`, `academic-essay-quote-verify`, and `academic-essay-export` respectively.
