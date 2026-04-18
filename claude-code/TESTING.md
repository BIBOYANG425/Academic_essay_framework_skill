# Claude Code workflow — manual tests

This file enumerates the golden paths, edge cases, and trigger prompts to walk before any significant edit. LLM outputs are non-deterministic, so these are manual-run checks, not a scripted suite.

## 1. Activation prompts per skill

For each of the 10 skills, in a fresh Claude Code session with no prior context, paste each prompt below and verify the expected skill activates (Claude invokes it via the Skill tool). If the skill does not activate, tighten the `description` field of that skill until it does.

### academic-essay (orchestrator)
- "Help me write a source-analysis essay about environmental justice."
- "I have an assignment due Friday, can we do it step by step?"
- "Run the essay workflow on this prompt."

### academic-essay-clarify-topic
- "My professor gave me this assignment prompt — can you help me understand what it's asking?"
- "I have a topic but I'm not sure what angle to take."

### academic-essay-collect-materials
- "I have the syllabus and two PDFs; where do I point you?"
- "Which course materials do you need to see first?"

### academic-essay-find-sources
- "I need more scholars to back up this argument — can you search for some?"
- "Find me primary-source candidates for this topic online."

### academic-essay-scholar-match
- "I have two scholars in mind, which one fits this primary source better?"
- "Is Winner the right framework for analyzing this feasibility study?"

### academic-essay-framework-check
- "Here's my outline — does it match the 4-paragraph structure?"
- "Run the framework check on this draft."

### academic-essay-quote-verify
- "Verify these quotes against the source PDF."
- "Is this quote word-for-word accurate on page 121?"

### academic-essay-draft
- "Write the first body paragraph using the verified quotes."
- "Draft the full essay from the approved outline."

### academic-essay-style-check
- "Check my draft for em dashes and banned phrases."
- "Audit this essay for corporate jargon overuse."

### academic-essay-export
- "Convert the final draft to a Word document."
- "Export this as a .docx with humanities formatting."

## 2. Golden paths

### 2a. Happy path, interactive
clarify → collect → find → scholar-match → framework-check → quote-verify → draft → style-check → export. Verify each stage produces its named output file in the workspace.

### 2b. Happy path, autoplan
Same stages, single command. Verify `autoplan-log.md` is produced and the taste gate surfaces any ambiguous calls.

### 2c. Loop-back
After `draft.md` is written, manually add a new quote to it. Re-invoke `academic-essay`. Verify the orchestrator re-runs `quote-verify` before `style-check`.

### 2d. Mid-flow resume
Interrupt a session after `scholars.md` is written. In a fresh Claude Code session, invoke `academic-essay` from the same workspace. Verify it offers "resume at framework-check."

### 2e. Redo stage
Ask the orchestrator to redo `scholar-match`. Verify the prior `scholars.md` is moved to `.archive/scholars-<timestamp>.md` and a new one replaces it.

### 2f. Skip stage
Tell the orchestrator to skip `find-sources`. Verify downstream stages succeed using only `materials.md`.

## 3. Edge cases

- Essay with zero quotes: `quote-verify` produces a clean audit showing no quotes to verify.
- Single unambiguous scholar candidate: autoplan proceeds without pausing.
- Two scholars tied at the top: autoplan pauses at the taste gate.
- Scanned image-only PDF: `quote-verify` fails cleanly asking for a text-layer version — does not silently succeed.
- `export` with `humanities-template.docx` missing: fails cleanly — does not silently produce an unformatted `.docx`.

## 4. Install smoke test

```bash
cp -r claude-code/academic-essay* ~/.claude/skills/
ls ~/.claude/skills/ | grep academic-essay | wc -l   # expect 10
```

Then in a fresh Claude Code session, ask "help me write an essay" and verify `academic-essay` activates.

## 5. Dog-food log

### 2026-04-17 — first end-to-end run

Live workflow run on a real argumentative-essay assignment (internship class, due same day): "Write on a political conflict or problem that may impact your personal life and/or career plans..." Student thesis: K-12 climate literacy as an incremental fix for two structural vulnerabilities (psychological distance + manufactured doubt) that enabled the 2026 federal green-policy rollback.

Full chain walked end-to-end — `clarify-topic → collect-materials → find-sources → scholar-match → framework-check (on draft) → quote-verify → draft → style-check → export`. Final output: 655-word essay (Times New Roman 12pt, double-spaced, page numbers) visually verified in Microsoft Word.

#### What worked

- **All 10 skills activated cleanly** via the Skill tool. No activation failures on any of the planned trigger prompts.
- **Skill registry hot-refreshed after `cp -r claude-code/academic-essay* ~/.claude/skills/`** — the running session picked up the new skills on the next tool call, no restart required. Same pattern held later when `academic-essay-style-check` was updated mid-session and reinstalled.
- **Export path survived the full pipeline.** `pandoc` + `humanities-template.docx` produced a valid Word 2007+ `.docx` with TNR 12pt, double-spacing, and a `PAGE` field footer. Re-export after a mid-session rewrite correctly archived the prior `essay.docx` under `.archive/essay-pre-colon-fix-<UTC>.docx`.
- **File-existence state machine held.** Every workspace file was written when and only when its stage finished; the orchestrator could resume by inspecting file existence at every hop.

#### Findings that drove changes in this session

- **Explanatory-colon rule missing from the style framework.** Visual QA of the first exported docx surfaced multiple explanatory colons (a sibling pattern to the em-dash ban that the original framework never closed). Added an `Explanatory colon ban` subsection to the Desktop canonical `SKILL.md`, propagated to the root copy and zip (three-copies invariant), and lifted into `academic-essay-style-check` with a scan step, dedicated output section, and an enumerative-list carve-out. Commit: `dd7c78c`. Re-audit + rewrite + re-export after the rule landed produced a clean final draft.

#### Findings deferred (for a follow-up commit)

- **Orchestrator decision table has no row for draft-first workflow.** When no `outline.md` is written, the table skips rows 5 and 6 and routes straight to `draft`, then to `style-check` — but `framework-check` on the draft has no automatic trigger. Had to invoke `framework-check` manually. Proposed fix: insert a row between current rows 8 and 9, `draft.md exists AND (framework-review.md missing OR older than draft.md) → framework-check`, so structural audit fires on the draft when no outline preceded it.
- **`academic-essay-clarify-topic`'s 7-item checklist is source-analysis-heavy.** For open-topic argumentative essays the load-bearing first question is "what's your political conflict or thesis direction?", currently lumped under item 3 (primary source type). The skill handled it gracefully because the student volunteered the topic in their opening message, but a pure argumentative-essay branch would ask topic/thesis up front. Proposed fix: add a short note — if the assignment is open-topic or argumentative, ask the topic as the first clarifying question before walking the standard 7-item checklist.
- **`academic-essay-quote-verify` with no source PDFs works but reads as ceremonial.** The skill produced a useful "N/A, paraphrase-based, URLs in `candidate-sources.md` are the practical verification path" audit, but the body is written as if source PDFs are always present. Proposed fix: add a short branch acknowledging the web-research / no-PDF case and what verification looks like there — URL reachability plus an author/year match against the cited claim.
- **`academic-essay-draft` preconditions are strict for a cold start.** With no outline and no prior `quote-verification.md`, the "clean or acknowledged" clause had to cover an empty state. Proposed fix: an explicit "cold-start path" note in the skill body — when no outline exists, the draft stage treats the empty quote-verification state as acknowledged-by-default, and the orchestrator's loop-back handles `quote-verify` after the draft surfaces any quotes.
- **Autoplan mode ran in practice as semi-interactive.** The user confirmed each stage even though auto mode was on, because none of the autoplan taste-gate conditions fired. Worth a real autoplan end-to-end on a future run to exercise the gate (e.g., on an essay where `scholar-match` has two tied candidates, or where `find-sources` returns 20+ hits).

#### Workspace state at end of run

At `/tmp/dogfood-essay/`:

```
topic.md              materials.md         candidate-sources.md
scholars.md           draft.md (655 words) quote-verification.md
framework-review.md   style-audit.md       essay.docx (12,978 bytes)
.archive/draft-20260418T005521Z.md
.archive/essay-pre-colon-fix-20260418T005600Z.docx
```
