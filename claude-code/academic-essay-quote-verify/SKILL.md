---
name: academic-essay-quote-verify
description: Use whenever an outline or draft contains direct quotes with page numbers, before building analysis around them. Extracts text from source PDFs using pdftotext or Claude Code's native PDF Read, word-for-word compares each quote to the source, and flags mismatches with the correct wording and page number. Preserves the word-for-word verification habit from the original academic-essay-framework skill (the Winner page-121 misquote example). Trigger aggressively when quotes appear in essay work. Produces quote-verification.md with per-quote status and corrections.
---

# academic-essay-quote-verify

## What this skill does

This is the quote-verification stage of the academic essay workflow. It takes every direct quote that appears in the user's outline or draft and verifies it word-for-word against the source PDF recorded in `materials.md`. Verification happens before analysis gets built around a quote, because analysis hung on a misquote has to be rewritten once the real wording surfaces. This skill is not a drafting skill and not a structural check — it only checks that the quoted text matches the source at the cited page. Drafting lives in `academic-essay-draft`, structural review lives in `academic-essay-framework-check`, and style audit lives in `academic-essay-style-check`. Output is `quote-verification.md` in the workspace.

## Quote verification

Verify every quote word-for-word against the original PDF before giving feedback or writing analysis around it. Search the source, find the exact page, and confirm the quote matches. If it does not, flag the correct wording and the correct page number clearly.

This has caught real misattributions in past work. For example, a Winner quote presented as "embody specific forms of authority and domination" when the actual page 121 phrasing reads "embody specific forms of power and authority." Always check before building analysis. The habit also catches paraphrases that have been promoted to direct quotes, which weaken the essay when a reader checks the citation.

When the primary source is available as a PDF (uploaded to the conversation, attached to project knowledge, or present in the working directory), search it directly. Use whatever extraction path the environment provides: the built-in PDF reading in Claude Desktop, or `pdftotext` / Claude Code's native PDF Read support when running in Claude Code. Confirm the exact wording from the extracted text. When the source is an outside scholar, verify against the primary text rather than a secondary summary.

## Extraction path

Pick up the source PDF paths from `materials.md`, which the earlier `academic-essay-collect-materials` stage wrote into the workspace. For each quote, open the source PDF at the cited page and compare the wording character by character. Try the extraction tools in this order:

1. **`pdftotext` via Bash.** Fastest for text-layer PDFs. Run `pdftotext -layout -f <page> -l <page> <source.pdf> -` and diff the output against the quote. Prefer this when the PDF has a real text layer.
2. **Claude Code's native PDF Read.** Fall back here when `pdftotext` is unavailable, when the text layer is messy, or when the quote spans a page break that `-layout` mangles.
3. **Fail cleanly on image-only or encrypted PDFs.** If both extraction paths come back empty or unreadable (scanned image-only PDF, encrypted PDF, missing text layer), do not silently mark the quote verified. Write a row with status `unverifiable` and a specific reason ("image-only PDF, needs OCR" or "PDF is encrypted, needs unlocked copy"), and ask the user for a text-layer version or an OCR'd copy before moving on.

Do not guess the wording from memory, and do not verify against a secondary summary or a Google Books snippet when the primary PDF is in hand.

## Output file: `quote-verification.md`

Write `quote-verification.md` in the workspace. The file opens with a one-line header, then the per-quote audit table, then a short prose section noting any corrections the user needs to make in the outline or draft. The exact file structure (strip the fence markers when writing into `quote-verification.md`).

```markdown
# Quote verification

## Per-quote audit

| Quote as written | Source path | Page | Verified? | Corrected wording (if any) |
|---|---|---|---|---|
| "embody specific forms of authority and domination" | sources/winner-artifacts.pdf | 121 | No | "embody specific forms of power and authority" |
| (next quote) | (path) | (page) | Yes / No / unverifiable | (correction or reason) |

## Corrections to apply

Prose notes on which quotes in the outline or draft need to be updated, what the corrected wording is, and whether any page number was also wrong. If a quote turned out to be a paraphrase promoted to a direct quote, flag that explicitly so the user can either restore the quotation marks on the real wording or demote the paraphrase back to indirect language.
```

Fill in every row of the audit table. Do not collapse multiple quotes into a single row, and do not omit a quote because it "looks right" — every quote gets checked against the PDF. If the outline has zero direct quotes, write the table with a single row reading "no quotes to verify" and mark the status accordingly; do not skip writing the file.

## Dates and provenance on archival documents

When the essay is source-analysis of an archival document, verify not just the quotes but the dates and provenance. Catching a 1969 document versus a 1975 document can reshape the entire argument, as can catching which parcel, box, or folder a given document belongs to. Add a short note under the audit table flagging any date or provenance mismatch found along the way, even if the quoted wording itself checks out.

## Termination condition

Stop and hand off once both are true:

- Every quote in the outline or draft has a row in the audit table, with status `Yes`, `No`, or `unverifiable` and (where needed) the corrected wording.
- `quote-verification.md` is written to the workspace.

Once the audit file is written, tell the user it is saved and name any quotes that need correcting before drafting continues. If the user revises the outline or draft in a way that adds, removes, or changes a quote, re-run this skill and overwrite `quote-verification.md`. Do not begin drafting, structural review, or style editing — those are the jobs of `academic-essay-draft`, `academic-essay-framework-check`, and `academic-essay-style-check`.
