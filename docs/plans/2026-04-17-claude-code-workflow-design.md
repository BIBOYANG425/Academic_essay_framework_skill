# Claude Code Workflow — Design Doc

**Date:** 2026-04-17
**Topic:** Decompose the Academic Essay Framework into a fine-grained Claude Code workflow, separated from the Desktop `.skill` distribution.

## Context

The repository currently ships a single Claude skill in two formats from one source of truth: `academic-essay-framework/SKILL.md`. The Desktop `.skill` zip bundle and the Claude Code directory install point at the same file. This is brittle when the two environments diverge (Desktop has `/mnt/skills/public/docx/`, Claude Code has shell tools like pandoc) and ties the two distributions together.

This design separates the two distributions completely and turns the Claude Code side into a fine-grained workflow of 10 chained sub-skills with an orchestrator and an autoplan mode.

## Goals

1. The Desktop `.skill` bundle stays as a single monolithic skill (unchanged behavior for existing users).
2. The Claude Code distribution becomes a workflow of 10 specialized skills that can be invoked end-to-end or individually.
3. State lives on disk, so stages survive context compression and users can resume mid-flow.
4. Autoplan mode runs the full chain with minimal interruption, pausing only at taste decisions.
5. Each sub-skill is independently useful outside the essay workflow (e.g., `quote-verify` works in any writing context).

## Non-goals

- Preserving a shared `SKILL.md` source between Desktop and Claude Code.
- Supporting non-humanities/social-science essay styles in the initial rollout.
- Deterministic regression testing of LLM-produced output.

## Architecture

Repo layout after separation:

```
academic-essay-framework-repo/
├── desktop/
│   ├── academic-essay-framework/SKILL.md
│   └── academic-essay-framework.skill
├── claude-code/
│   ├── academic-essay/SKILL.md                     (orchestrator)
│   ├── academic-essay-clarify-topic/SKILL.md
│   ├── academic-essay-collect-materials/SKILL.md
│   ├── academic-essay-find-sources/SKILL.md
│   ├── academic-essay-scholar-match/SKILL.md
│   ├── academic-essay-framework-check/SKILL.md
│   ├── academic-essay-quote-verify/SKILL.md
│   ├── academic-essay-draft/SKILL.md
│   ├── academic-essay-style-check/SKILL.md
│   └── academic-essay-export/
│       ├── SKILL.md
│       └── humanities-template.docx
├── README.md
├── CLAUDE.md
└── LICENSE
```

Sub-skills use the `academic-essay-<stage>` naming convention so they sort together in `~/.claude/skills/` and are discoverable as a family. The orchestrator is `academic-essay` (no suffix) — that is the entry point users invoke.

Only the orchestrator knows the chain order. Every sub-skill is standalone: it does one thing, reads named input files, writes one named output file, does not reference any other skill by name.

## Components

Ten skills. For each: trigger description, single responsibility, inputs, output file.

**`academic-essay` (orchestrator).** Triggers when user asks for essay help, pastes an assignment prompt, or invokes the workflow directly. Reads the workspace state, picks the next stage, invokes the corresponding sub-skill, handles autoplan mode, manages loop-backs when a later stage surfaces work for an earlier stage. Inputs: user prompt, current workspace. Output: invokes the right sub-skill; in autoplan mode, produces a final-gate summary in `autoplan-log.md`.

**`academic-essay-clarify-topic`.** Triggers on assignment prompt or topic with no prior workspace. Asks thoughtful questions one at a time about discipline, lens, source type, length, due date, professor expectations. Output: `topic.md`.

**`academic-essay-collect-materials`.** Triggers when `topic.md` exists but `materials.md` does not. Prompts user to point Claude at syllabus, assignment brief, required readings, past professor feedback; indexes file paths in the working directory without copying. Output: `materials.md` (catalogue of paths).

**`academic-essay-find-sources`.** Triggers when user wants additional sources beyond what they already have. Web-searches candidate scholars and primary-source leads using Claude Code's WebSearch/WebFetch. Output: `candidate-sources.md` (ranked shortlist with citations and rationale).

**`academic-essay-scholar-match`.** Triggers when user has a primary source plus candidate scholars and wants to pick the tight fit. Evaluates each scholar's framework against the primary source's mechanics (the Winner / Roy / Bullard / Pulido matching logic from the original SKILL.md). Output: `scholars.md` (chosen scholars + justifications + rejected candidates + reasons).

**`academic-essay-framework-check`.** Triggers when user shares an outline, or after `draft` is written. Runs the 4-paragraph outline review table (hook/bridge/thesis, argument/evidence/analysis, restate/so-what), flags specific gaps. Output: `framework-review.md`.

**`academic-essay-quote-verify`.** Triggers when an outline or draft contains quotes with page numbers. Extracts source PDF text via `pdftotext` or Claude Code's native PDF Read; word-for-word checks each quote; flags mismatches with corrected wording and correct page. Output: `quote-verification.md`.

**`academic-essay-draft`.** Triggers when framework check and quote verification are clean and user is ready to write. Writes the full essay following voice rules (historical, grounded, argumentative) and the approved outline, using only verified quotes. Output: `draft.md`.

**`academic-essay-style-check`.** Triggers on an existing draft. Scans for em dashes, banned phrases, and restricted-word overuse (two-uses-per-cluster cap); flags with specific replacements; rewrites if user approves. Output: `style-audit.md`.

**`academic-essay-export`.** Triggers when style-check is clean and user is ready to submit. Runs `pandoc draft.md -o essay.docx --reference-doc=humanities-template.docx` to produce a Word document with double-spaced Times New Roman 12pt text and page numbers. Ships `humanities-template.docx` in the skill directory (pandoc cannot set font or line-spacing via CLI flags — a reference template is required). Output: `essay.docx`.

## Data flow

**The workspace is a per-essay directory.** User picks a folder for each essay (e.g., `essays/environmental-justice-final/`) and runs Claude Code there, or points the orchestrator at it. Everything one essay needs lives in that one folder.

**Each stage writes exactly one output file**, named predictably:

```
essays/my-essay/
├── topic.md               ← clarify-topic
├── materials.md           ← collect-materials (paths only, no copies)
├── candidate-sources.md   ← find-sources
├── scholars.md            ← scholar-match
├── outline.md             ← user-provided (or drafted with help)
├── framework-review.md    ← framework-check
├── quote-verification.md  ← quote-verify
├── draft.md               ← draft
├── style-audit.md         ← style-check
├── essay.docx             ← export
└── autoplan-log.md        ← autoplan mode only
```

**User PDFs stay where they are.** `collect-materials` records existing paths in `materials.md`; it does not copy files into the workspace. Downstream stages read PDFs from their original locations.

**Sub-skills read only what they need.** Examples: `scholar-match` reads `topic.md` + `materials.md` + `candidate-sources.md`, writes `scholars.md`. `quote-verify` reads `outline.md` (or `draft.md`) + `materials.md`, writes `quote-verification.md`. `draft` reads all upstream outputs, writes `draft.md`. `style-check` reads `draft.md`, writes `style-audit.md`.

**The orchestrator decides the next stage by file existence.** Its `SKILL.md` contains an explicit decision table: if `topic.md` missing → `clarify-topic`; if `materials.md` missing → `collect-materials`; if `draft.md` is newer than `quote-verification.md` → re-run `quote-verify`; and so on. No separate state file — the workspace is the state.

**Re-runs overwrite.** When a later stage triggers a re-run of an earlier one (e.g., `draft` introduces a new quote and orchestrator re-invokes `quote-verify`), the prior output file is overwritten. Git handles history if the user commits the workspace.

**Autoplan mode.** The orchestrator accepts a mode flag ("run end-to-end" or a dedicated slash command). In autoplan, sub-skills receive `auto=true`, skip user prompts for unambiguous calls, and log every auto-decision to `autoplan-log.md`. At the end, the orchestrator shows the log plus the draft to the user for a single final-gate approval.

## Error handling, taste gate, mid-flow resumption

**Stage failure contract.** A sub-skill writes its output file only when its work is complete. Partial runs write nothing. The orchestrator's "what's done" logic (file existence) remains correct; a crashed stage leaves no file and re-running starts that stage from scratch without polluting downstream inputs.

**What pauses autoplan (taste gate).** Autoplan runs silently unless it hits one of these, at which point it surfaces the call to the user:

- `scholar-match`: more than one candidate tied at the top, or no candidate above the floor.
- `quote-verify`: a mismatch that requires judgment (paraphrase-to-direct conversion, page number off by 1-2, punctuation or emphasis difference).
- `framework-check`: a structural gap with more than one valid fix (e.g., "argument too broad: split across body paragraphs, OR consolidate evidence").
- `find-sources`: more than roughly 20 candidates returned (too broad, narrow the query first).
- `export`: would overwrite an existing `essay.docx`.

Anything unambiguous auto-proceeds and logs the decision with reasoning. Final gate: user sees the log and the draft, approves or sends back.

**Mid-flow resumption.** When the orchestrator is invoked in a workspace that already contains files, it reports the current stage (inferred from file existence and mtimes) and offers: resume at the next stage, redo a prior stage, or restart from scratch. Redo archives the existing output to `.archive/<stage>-<timestamp>.md` before overwriting. Restart archives the full workspace under `.archive/<timestamp>/`.

**Skip a stage.** User can tell the orchestrator to skip (for example, "skip `find-sources`, I have my scholars already"). The orchestrator requires downstream inputs to exist in some form; if the user skips `collect-materials` but later stages cannot find the PDFs, those stages fail clearly at read time. No silent degradation.

**External dependency failures.**

- `pandoc` missing when `export` runs: fail with the install command for the user's platform; do not silently produce an unformatted `.docx`.
- `pdftotext` missing: `quote-verify` falls back to Claude Code's native PDF Read. If the PDF is image-only or encrypted, fail with a clear message asking for a text-layer version.
- WebSearch unavailable: `find-sources` reports the condition and asks the user to paste a scholar shortlist, rather than silently skipping.

## Testing and validation

**Dog-food on a real assignment.** The ground-truth test is running the full chain end-to-end on a past essay (one with a known grade) or a live one. Set up a fresh workspace, run `academic-essay` from scratch, compare Claude's draft against the submitted version. Where they diverge reveals whether the rules still have bite after decomposition. Run this at least once before declaring the workflow ready.

**Manual golden-path checklist.** A short `claude-code/TESTING.md` enumerates paths to walk before any significant edit:

- Happy path, interactive mode: clarify → collect → find → scholar-match → framework-check → quote-verify → draft → style-check → export.
- Happy path, autoplan: same stages, single command, verify taste gate surfaces where expected.
- Loop-back: after draft, add a new quote manually, re-invoke orchestrator, verify it re-runs `quote-verify` before `style-check`.
- Mid-flow resume: interrupt after `scholars.md` is written, relaunch orchestrator in a new session, verify it offers "resume at framework-check."
- Redo stage: redo `scholar-match`, verify prior file archived.
- Skip stage: skip `find-sources`, verify downstream stages read from `materials.md`.

**Trigger-description verification.** The biggest failure mode for a Claude Code skill is a description that does not fire. For each of the 10 skills, write 2-3 sample user prompts that should trigger it. In a clean Claude Code session with no prior context, paste each prompt and verify Claude invokes the right skill. Tighten the `description` field on any skill that does not activate. Keep the prompts in `claude-code/TESTING.md` so they are re-runnable.

**Install smoke test.**

```bash
cp -r claude-code/* ~/.claude/skills/
ls ~/.claude/skills/ | grep academic-essay   # expect 10 lines
```

Then in a fresh Claude Code session, run `/academic-essay` or ask "help me write an essay" and verify the orchestrator activates.

**Edge cases in TESTING.md.** Essay with zero quotes; one unambiguous scholar (no gate); two ambiguous scholars (gate triggers); scanned-image PDF (quote-verify fails cleanly with a useful error); `export` when `humanities-template.docx` is missing (fails cleanly, does not silently produce unformatted docx).

**Explicitly not tested.** Snapshot regression against expected LLM outputs (non-deterministic, noise exceeds signal). Mocked unit tests per skill (a skill that passes mocks and fails in a real session is worse than no tests).

## Open questions for the implementation plan

1. Should the orchestrator be invokable as a slash command (`/essay`, `/essay-auto`) as well as a skill, or skill-only?
2. Location of `humanities-template.docx` inside `academic-essay-export/` — is there a convention in Claude Code for shipping binary assets alongside a SKILL.md?
3. How should the orchestrator surface autoplan's final approval gate — inline summary, a separate doc, or both?
4. Fate of the existing root-level `academic-essay-framework-repo.zip` (a manually created snapshot) once the repo restructures into `desktop/` and `claude-code/` subdirectories.

These decisions belong in the implementation plan, not this design.
