---
name: academic-essay-find-sources
description: Use when the user wants to find additional scholars or primary-source candidates beyond what they already have catalogued in materials.md. Runs web searches using Claude Code's WebSearch and WebFetch tools, returns a ranked shortlist with citations. Trigger when the user asks for more sources, needs leads on scholars, or wants primary-source candidates for a topic. Produces candidate-sources.md.
---

# academic-essay-find-sources

## What this skill does

This is the third stage of the academic essay workflow. It runs after `academic-essay-collect-materials` has recorded the user's on-disk materials in `materials.md`. Use this skill to discover NEW scholarly sources and primary-source candidates the user does not yet have — the ones that fill gaps in the existing pile. This skill runs web searches with Claude Code's `WebSearch` tool to generate queries and `WebFetch` tool to inspect promising results. It does not verify quotes (that is `academic-essay-quote-verify`'s job) and it does not draft anything. It produces a single `candidate-sources.md` shortlist in the workspace.

## Read `topic.md` first

Before firing any search, read `topic.md` to pin down the discipline, lens, and primary-source type. The search queries you generate must reflect that lens. A "civil rights" query for a paper framed around environmental justice is a miss. If `topic.md` is missing, stop and tell the user to run `academic-essay-clarify-topic` first.

## Search preference order

Prefer these surfaces, in this order:

1. **JSTOR** — peer-reviewed humanities and social-science journals.
2. **Google Scholar** — broader academic coverage, citation counts, related-work links.
3. **University press catalogues** — Harvard, Princeton, Chicago, Yale, Oxford, Cambridge, MIT, California.
4. **Archival collections** — Library of Congress, National Archives, university special collections, Internet Archive scholarly holdings.

Use `WebSearch` to generate queries and collect candidate URLs. Use `WebFetch` to inspect the abstract, table of contents, or finding aid on promising hits before recording them. Skip anything behind an unresolvable paywall when the abstract does not give you enough to judge relevance.

## Cap the shortlist

Cap the shortlist at roughly 15 candidates. If the topic is so broad that 15 is not enough, stop and ask the user to narrow the lens — do not dump 50 loosely-related sources on them. If under 15 is plenty for the scope, stop earlier.

## Output file: `candidate-sources.md`

Write `candidate-sources.md` in the workspace with two H2 sections — `Scholars` and `Primary sources`. Every entry has name/title, key work, framework or relevance, and a one-line reason it might fit the essay.

The exact file contents (strip the fence markers when writing).

```markdown
# Candidate sources

## Scholars
- **Scholar name** — *Key work (Year)* — Framework / relevance — Why it might fit this essay.
- **Scholar name** — *Key work (Year)* — Framework / relevance — Why it might fit this essay.

## Primary sources
- **Document title** — Collection / archive — Type (policy document, photo, feasibility study, etc.) — Why it might fit this essay.
- **Document title** — Collection / archive — Type (policy document, photo, feasibility study, etc.) — Why it might fit this essay.
```

If either section has no candidates, write `None` as the entire body of that section instead of a bullet list. Keep both headings so downstream skills can see the section was considered.

## Do not verify quotes

Do not pull quotes from these candidates and do not cross-check page numbers here. Quote verification happens in `academic-essay-quote-verify` after the user picks which candidates to actually use. Your job is leads, not proofs.

## Termination condition

Stop searching and write `candidate-sources.md` as soon as either is true:

- The user says some version of "that's enough" / "stop searching" / "write it up."
- You have reached roughly 15 candidates AND the user approves the shortlist.

After writing, tell the user the file is saved and that the next skill in the workflow (`academic-essay-scholar-match`) will take it from here. Do not begin matching scholars to primary sources, verifying quotes, or drafting — those are downstream skills' jobs.
