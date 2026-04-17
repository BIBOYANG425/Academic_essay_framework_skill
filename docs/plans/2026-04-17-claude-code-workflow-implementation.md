# Academic Essay Framework Claude Code Workflow — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use `superpowers:executing-plans` to implement this plan task-by-task.

**Goal:** Build a 10-skill Claude Code workflow (1 orchestrator + 9 sub-skills) that decomposes the existing monolithic SKILL.md into fine-grained stages, while keeping the Desktop `.skill` distribution unchanged.

**Architecture:** Orchestrator + standalone subs, per `docs/plans/2026-04-17-claude-code-upgrade-design.md`. Each SKILL.md is a Markdown file with YAML frontmatter (`name`, `description`). Sub-skills communicate via named output files in a per-essay workspace — file existence is the state. Only the orchestrator knows chain order; sub-skills never reference each other by name.

**Tech Stack:** Markdown + YAML for skills; shell (`pandoc`, `pdftotext`, `zip`) for tooling; Claude Code built-in Read/Grep/Bash/WebSearch/WebFetch for skill I/O.

**Branch:** Work on `claude-code-upgrade`. Keep `main` clean until restructure is proven end-to-end.

**Source of truth for content:** The existing `academic-essay-framework/SKILL.md` contains all the framework rules, scholar-matching logic, quote-verification habits, and style prohibitions. Each sub-skill's body lifts and narrows the relevant section(s). The plan specifies frontmatter (which must be exact, since it controls activation) and points at which SKILL.md sections to lift for each body. Do not rewrite the rules from scratch — preserve the specific examples (Winner page-121 misquote, Roy / Bullard / Pulido scholar-source pairings) verbatim.

---

## Phase 1: Repo restructure

### Task 1: Create feature branch

**Files:** none

**Step 1:** Create and switch to branch.
```bash
git checkout -b claude-code-upgrade
git status
```
Expected: "On branch claude-code-upgrade, nothing to commit, working tree clean"

### Task 2: Move Desktop skill into `desktop/` subdir

**Files:**
- Move: `academic-essay-framework/` → `desktop/academic-essay-framework/`
- Move: `academic-essay-framework.skill` → `desktop/academic-essay-framework.skill`
- Move: `academic-essay-framework-SKILL.md` → `desktop/academic-essay-framework-SKILL.md`
- Move: `academic-essay-framework-repo.zip` → `desktop/academic-essay-framework-repo.zip` (stays stale — regenerated later if the user wants)

**Step 1:** Create `desktop/` and move files with `git mv` to preserve history.
```bash
mkdir -p desktop
git mv academic-essay-framework desktop/academic-essay-framework
git mv academic-essay-framework.skill desktop/academic-essay-framework.skill
git mv academic-essay-framework-SKILL.md desktop/academic-essay-framework-SKILL.md
git mv academic-essay-framework-repo.zip desktop/academic-essay-framework-repo.zip
git status --short
```
Expected: 4 renames shown.

**Step 2:** Verify the `.skill` zip still extracts cleanly after the move (zip path inside the archive is unchanged; only its container path moved).
```bash
unzip -l desktop/academic-essay-framework.skill
```
Expected: one entry `academic-essay-framework/SKILL.md`.

**Step 3:** Re-verify the three-copies-in-sync invariant still holds post-move.
```bash
diff desktop/academic-essay-framework/SKILL.md desktop/academic-essay-framework-SKILL.md && echo SYNCED
```
Expected: "SYNCED".

**Step 4:** Commit.
```bash
git commit -m "refactor: move Desktop skill into desktop/ subdir"
```

### Task 3: Scaffold `claude-code/` directory skeleton

**Files:**
- Create: `claude-code/academic-essay/SKILL.md` (stub)
- Create: `claude-code/academic-essay-clarify-topic/SKILL.md` (stub)
- Create: `claude-code/academic-essay-collect-materials/SKILL.md` (stub)
- Create: `claude-code/academic-essay-find-sources/SKILL.md` (stub)
- Create: `claude-code/academic-essay-scholar-match/SKILL.md` (stub)
- Create: `claude-code/academic-essay-framework-check/SKILL.md` (stub)
- Create: `claude-code/academic-essay-quote-verify/SKILL.md` (stub)
- Create: `claude-code/academic-essay-draft/SKILL.md` (stub)
- Create: `claude-code/academic-essay-style-check/SKILL.md` (stub)
- Create: `claude-code/academic-essay-export/SKILL.md` (stub)
- Create: `claude-code/TESTING.md` (stub)

**Step 1:** Create all 10 skill directories.
```bash
mkdir -p claude-code
for s in academic-essay academic-essay-clarify-topic academic-essay-collect-materials academic-essay-find-sources academic-essay-scholar-match academic-essay-framework-check academic-essay-quote-verify academic-essay-draft academic-essay-style-check academic-essay-export; do
  mkdir -p "claude-code/$s"
done
```

**Step 2:** Place a one-line placeholder SKILL.md in each, with frontmatter present so the tree is git-trackable:
```bash
for s in academic-essay academic-essay-clarify-topic academic-essay-collect-materials academic-essay-find-sources academic-essay-scholar-match academic-essay-framework-check academic-essay-quote-verify academic-essay-draft academic-essay-style-check academic-essay-export; do
  cat > "claude-code/$s/SKILL.md" <<EOF
---
name: $s
description: STUB — populated by a later task. Do not install yet.
---

# $s

Stub. See implementation plan task for $s.
EOF
done
```

**Step 3:** Create empty `claude-code/TESTING.md` placeholder.
```bash
echo "# TESTING.md — populated by Task 4." > claude-code/TESTING.md
```

**Step 4:** Commit.
```bash
git add claude-code/
git commit -m "scaffold: claude-code/ directory with 10 skill stubs and TESTING.md"
```

---

## Phase 2: Activation harness + first real skill

### Task 4: Write TESTING.md with trigger prompts

**Files:**
- Modify: `claude-code/TESTING.md`

**Step 1:** Write TESTING.md with three sections. Content:

```markdown
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
```

**Step 2:** Commit.
```bash
git add claude-code/TESTING.md
git commit -m "docs: write TESTING.md with activation prompts and golden paths"
```

### Task 5: Populate `academic-essay-clarify-topic` SKILL.md

**Files:**
- Modify: `claude-code/academic-essay-clarify-topic/SKILL.md`

**Step 1:** Write the SKILL.md. Frontmatter (exact):

```yaml
---
name: academic-essay-clarify-topic
description: Use when the user drops an academic essay assignment prompt, topic idea, or course brief and no essay workspace has been started yet. Asks thoughtful clarifying questions one at a time to pin down discipline, lens, primary source type, length, due date, professor expectations, and prior feedback. Trigger aggressively whenever the user shares an assignment they have not yet scoped. Produces topic.md in the workspace.
---
```

Body outline (write in full):
- What this skill does (1 paragraph)
- The clarifying question checklist: discipline, lens, primary source type and availability, length, due date, professor expectations from prior assignments, any grade received on similar prior work
- Rule: ask one question per message, prefer multiple-choice when possible
- Output file shape: `topic.md` with sections `Assignment summary`, `Discipline and lens`, `Primary source(s)`, `Length and due date`, `Professor expectations`, `Prior feedback`
- Termination condition: user says "that's enough, let's move on" OR all checklist items have an answer — then write `topic.md` and stop

**Step 2:** Smoke-test activation. Stage, do not commit yet.
```bash
git add claude-code/academic-essay-clarify-topic/SKILL.md
# Then: in a fresh Claude Code session with claude-code/academic-essay-clarify-topic/ installed,
# paste each of the 2 activation prompts from TESTING.md and verify Claude invokes the skill.
```

Expected: skill activates on both prompts. If not, tighten `description` — add more trigger phrases or sharpen the "when to use" clause.

**Step 3:** Commit.
```bash
git commit -m "feat(claude-code): add academic-essay-clarify-topic skill"
```

---

## Phase 3: Input-stage sub-skills

### Task 6: `academic-essay-collect-materials`

**Files:**
- Modify: `claude-code/academic-essay-collect-materials/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-collect-materials
description: Use after topic.md exists but before any source search or analysis happens. Prompts the user to point Claude at syllabus, assignment brief, required readings, past professor feedback, and any other course materials. Indexes the file paths in the working directory without copying. Trigger when the user is ready to gather inputs for an essay in progress, or when topic.md is present but materials.md is not. Produces materials.md cataloguing paths and brief notes per file.
---
```

Body outline:
- Ask the user to name each material and provide its path (in working dir or elsewhere on disk). One at a time.
- For each file: verify the path exists (`ls` or Read), note its type (PDF, docx, image), capture a one-line description from the user or derive from filename.
- Structure of `materials.md`: one section per category (Syllabus, Assignment brief, Required readings, Optional readings, Professor feedback), each entry `- path — description`.
- Do not copy files into the workspace.
- Terminates when the user says "that's all" or equivalent.

**Step 2:** Activation smoke test against TESTING.md prompts.

**Step 3:** Commit: `feat(claude-code): add academic-essay-collect-materials skill`

### Task 7: `academic-essay-find-sources`

**Files:**
- Modify: `claude-code/academic-essay-find-sources/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-find-sources
description: Use when the user wants to find additional scholars or primary-source candidates beyond what they already have catalogued in materials.md. Runs web searches using Claude Code's WebSearch and WebFetch tools, returns a ranked shortlist with citations. Trigger when the user asks for more sources, needs leads on scholars, or wants primary-source candidates for a topic. Produces candidate-sources.md.
---
```

Body outline:
- Read `topic.md` for the lens and discipline before searching.
- Prefer academic sources: JSTOR, Google Scholar, university press catalogues, archival collections.
- Return at most ~15 candidates; if more would be relevant, narrow by asking the user.
- Output `candidate-sources.md` with two sections: `Scholars` (each entry: name, key work, framework, why it might fit) and `Primary sources` (each entry: title, date, provenance, relevance).
- Do not verify quotes at this stage; that is `academic-essay-quote-verify`'s job.

**Step 2:** Activation smoke test.

**Step 3:** Commit: `feat(claude-code): add academic-essay-find-sources skill`

### Task 8: `academic-essay-scholar-match`

**Files:**
- Modify: `claude-code/academic-essay-scholar-match/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-scholar-match
description: Use when evaluating whether a scholar's framework tightly matches the mechanics of the primary source an essay analyzes. Preserves the scholar-source matching logic from the original academic-essay-framework skill (Winner on design encoding politics, Roy on sovereign planning power, Bullard and Pulido on environmental burdens by race and class). Trigger when the user has a candidate scholar and a primary source and is deciding whether the pairing holds, or when multiple candidates compete. Produces scholars.md with the chosen scholar, justification, and rejected candidates with reasons.
---
```

Body: lift verbatim from the "Scholar-source mechanics matching" section of `desktop/academic-essay-framework/SKILL.md`. Preserve the four worked examples (slow violence mismatch, Winner architecture fit, Roy commissioned study fit, Bullard/Pulido racial burden fit). Add an output-file section specifying `scholars.md` structure.

**Step 2:** Activation smoke test.

**Step 3:** Commit: `feat(claude-code): add academic-essay-scholar-match skill`

---

## Phase 4: Analysis-stage sub-skills

### Task 9: `academic-essay-framework-check`

**Files:**
- Modify: `claude-code/academic-essay-framework-check/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-framework-check
description: Use when the user shares an outline or a draft and wants it checked against the 4-paragraph academic-essay framework (intro with hook/bridge/thesis, two body paragraphs each with argument/evidence/analysis, conclusion with restate/so-what). Produces a framework-vs-outline review table flagging specific gaps, historical inversions, or overbroad arguments. Trigger when the user asks to check an outline, run the framework check, or audit essay structure. Produces framework-review.md.
---
```

Body: lift the "4-paragraph framework" + "Outline review workflow" sections from `desktop/academic-essay-framework/SKILL.md` verbatim, including the review table template. Add a section specifying the output file `framework-review.md` that contains the filled-out table plus prose feedback.

**Step 2:** Activation smoke test.

**Step 3:** Commit: `feat(claude-code): add academic-essay-framework-check skill`

### Task 10: `academic-essay-quote-verify`

**Files:**
- Modify: `claude-code/academic-essay-quote-verify/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-quote-verify
description: Use whenever an outline or draft contains direct quotes with page numbers, before building analysis around them. Extracts text from source PDFs using pdftotext or Claude Code's native PDF Read, word-for-word compares each quote to the source, and flags mismatches with the correct wording and page number. Preserves the word-for-word verification habit from the original academic-essay-framework skill (the Winner page-121 misquote example). Trigger aggressively when quotes appear in essay work. Produces quote-verification.md with per-quote status and corrections.
---
```

Body: lift the "Quote verification" section from `desktop/academic-essay-framework/SKILL.md` verbatim, including the Winner page-121 misquote example. Specify the extraction order: try `pdftotext` first, fall back to Claude Code's native PDF Read, fail clearly on image-only PDFs. Specify `quote-verification.md` output structure: one row per quote with columns `quote as written | source path | page | verified? | corrected wording (if any)`.

**Step 2:** Activation smoke test.

**Step 3:** Commit: `feat(claude-code): add academic-essay-quote-verify skill`

---

## Phase 5: Output-stage sub-skills

### Task 11: `academic-essay-draft`

**Files:**
- Modify: `claude-code/academic-essay-draft/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-draft
description: Use when framework-review.md and quote-verification.md are clean (or their issues acknowledged) and the user is ready to write the full essay draft. Writes the essay following the voice rules from the original academic-essay-framework skill (historical, grounded, argumentative, varied sentence rhythm), using only verified quotes. Trigger when the user asks to draft, write, or build the essay after outline and evidence are settled. Produces draft.md.
---
```

Body: lift the "Writing style for drafts and revisions" section from `desktop/academic-essay-framework/SKILL.md` verbatim (but without the em-dash / banned-phrases / restricted-words subsections — those belong to `academic-essay-style-check`). Add a rule: read `topic.md`, `scholars.md`, outline (if present), and `quote-verification.md` before writing. Do not introduce new quotes that haven't been verified — if a new quote is needed mid-draft, stop and flag for a re-run of `quote-verify`. Output: `draft.md` as the full essay.

**Step 2:** Activation smoke test.

**Step 3:** Commit: `feat(claude-code): add academic-essay-draft skill`

### Task 12: `academic-essay-style-check`

**Files:**
- Modify: `claude-code/academic-essay-style-check/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay-style-check
description: Use when a draft.md exists and needs a style audit before export. Scans for em dashes (banned), banned phrases and overused transitions, buzzword clichés, and restricted-word overuse (corporate jargon cluster and vague-qualifier cluster capped at two uses per essay). Preserves the banned-phrase and restricted-word lists verbatim from the original academic-essay-framework skill. Trigger when the user asks to check style, audit jargon, or polish the draft. Produces style-audit.md with findings and suggested specific replacements.
---
```

Body: lift the "Em dash ban", "Banned phrases", and "Restricted words" subsections from `desktop/academic-essay-framework/SKILL.md` verbatim — preserve every banned-phrase and restricted-word list exactly. Specify the scan algorithm: grep for em dashes, grep for each banned phrase, count occurrences per restricted-word cluster against the two-uses cap. Output `style-audit.md` with sections `Em dashes found`, `Banned phrases found`, `Restricted words over the cap`, each entry citing the draft line and suggesting a concrete replacement.

**Step 2:** Activation smoke test.

**Step 3:** Commit: `feat(claude-code): add academic-essay-style-check skill`

### Task 13: `academic-essay-export` + bundled template

**Files:**
- Modify: `claude-code/academic-essay-export/SKILL.md`
- Create: `claude-code/academic-essay-export/humanities-template.docx`

**Step 1:** Produce `humanities-template.docx`. Generate from a minimal pandoc invocation, then open in Word or LibreOffice once to set Normal style to Times New Roman 12pt with double line spacing, and enable page numbers in the footer. Save. Alternative: use a pandoc `-s` template generated via `pandoc --print-default-data-file=reference.docx > humanities-template.docx` and edit the styles in Word.

The template only needs to define the default paragraph style — pandoc will apply it to all body text.

**Step 2:** Write the SKILL.md. Frontmatter (exact):
```yaml
---
name: academic-essay-export
description: Use when style-audit.md is clean and the user is ready to submit the essay. Converts draft.md to a Word document using pandoc with the bundled humanities-template.docx, producing double-spaced Times New Roman 12pt text with page numbers. Fails clearly if pandoc is missing or the template file is absent — does not silently produce unformatted output. Trigger when the user asks to export, convert to docx, or prepare for submission. Produces essay.docx in the workspace.
---
```

Body outline:
- Preconditions: `draft.md` exists and style-check has been passed (or its remaining issues acknowledged by the user).
- Check `pandoc` is on PATH: `command -v pandoc || fail with install instructions`.
- Check the bundled template is readable.
- Run: `pandoc draft.md -o essay.docx --reference-doc="$SKILL_DIR/humanities-template.docx"` where `$SKILL_DIR` resolves to this skill's directory.
- Warn and abort (not overwrite) if `essay.docx` already exists in the workspace, unless the user confirms.

**Step 3:** Smoke test: export on a trivial draft.md, open the .docx, verify Times New Roman 12pt, double-spaced, page numbers visible.

**Step 4:** Commit:
```bash
git add claude-code/academic-essay-export/
git commit -m "feat(claude-code): add academic-essay-export skill with humanities-template.docx"
```

---

## Phase 6: Orchestrator flow logic + autoplan

### Task 14: Populate orchestrator SKILL.md with decision table

**Files:**
- Modify: `claude-code/academic-essay/SKILL.md`

**Step 1:** Frontmatter (exact):
```yaml
---
name: academic-essay
description: Use whenever the user asks for help with an academic essay (humanities or social-science source-analysis or argumentative papers), pastes an assignment prompt, or explicitly invokes the essay workflow. Orchestrates a 10-stage workflow by inspecting the per-essay workspace directory, picking the next stage based on which output files exist, and invoking the corresponding sub-skill. Handles mid-flow resume, stage redo, stage skip, and loop-backs when a later stage surfaces work for an earlier stage. Trigger aggressively on any academic-writing help request.
---
```

**Step 2:** Body: three sections.

**Section A — The chain.** Ordered list of the 9 sub-skills with a one-line summary each.

**Section B — The decision table.** For each state of the workspace, which sub-skill to invoke next. Table columns: `Condition` | `Next stage` | `Why`. Rows:
- `topic.md missing` | `academic-essay-clarify-topic` | starting fresh
- `materials.md missing` | `academic-essay-collect-materials` | have a topic, need materials
- user wants more sources | `academic-essay-find-sources` | materials alone aren't enough
- `candidate-sources.md` exists and `scholars.md` missing | `academic-essay-scholar-match` | pick tight fit
- `outline.md` exists and `framework-review.md` missing or older | `academic-essay-framework-check` | check structure
- outline has quotes and `quote-verification.md` missing or older | `academic-essay-quote-verify` | verify before building analysis
- all above clean, no `draft.md` | `academic-essay-draft` | write
- `draft.md` exists and newer than `quote-verification.md` | `academic-essay-quote-verify` | loop-back on new quotes
- `draft.md` exists and `style-audit.md` missing or older | `academic-essay-style-check` | audit
- `style-audit.md` clean, no `essay.docx` | `academic-essay-export` | ship

**Section C — Mid-flow behavior.** When invoked in an existing workspace: report current state in one line, offer user `resume | redo <stage> | restart | skip <stage>`. Redo archives the stage's output to `.archive/<stage>-<UTC timestamp>.md` before overwriting. Restart archives the whole workspace to `.archive/<timestamp>/`.

**Step 3:** Activation smoke test against the 3 orchestrator prompts in TESTING.md.

**Step 4:** Commit: `feat(claude-code): orchestrator decision table and mid-flow behavior`

### Task 15: Add autoplan mode and taste gate

**Files:**
- Modify: `claude-code/academic-essay/SKILL.md`

**Step 1:** Add an "Autoplan mode" section to the orchestrator body.

- Triggered when the user says "run end-to-end", "full workflow", "autoplan", or similar.
- In autoplan, walk the chain stage by stage without prompting the user between stages.
- Each sub-skill receives an `auto=true` hint in the invocation; the sub-skill is expected to proceed silently when calls are unambiguous and pause (return to orchestrator) when not.
- Taste-gate conditions (pause autoplan):
  - `scholar-match`: more than one candidate tied at the top OR no candidate above the floor.
  - `quote-verify`: mismatch requiring judgment (paraphrase→direct conversion, page off by 1-2, punctuation/emphasis diff).
  - `framework-check`: gap with more than one valid fix.
  - `find-sources`: more than ~20 candidates returned.
  - `export`: would overwrite existing `essay.docx`.
- Every auto-decision appends to `autoplan-log.md` with the form:
  ```
  ## <UTC timestamp> — <stage>
  - Decision: <what was chosen>
  - Reason: <one-line why>
  ```
- Final gate: after `style-check` is clean and before `export` runs, show the user the contents of `autoplan-log.md` plus `draft.md` and ask for a single go/no-go.

**Step 2:** Commit: `feat(claude-code): orchestrator autoplan mode and taste-decision gate`

---

## Phase 7: Docs and integration

### Task 16: Update root README for new layout

**Files:**
- Modify: `README.md`

**Step 1:** Rewrite the Installation and Structure sections to reflect the `desktop/` vs `claude-code/` split.

- Installation — Desktop: download `desktop/academic-essay-framework.skill`, upload in Settings → Skills.
- Installation — Claude Code: `cp -r claude-code/academic-essay* ~/.claude/skills/` (user-wide) or into `<project>/.claude/skills/` (project-scoped). Note that there are 10 skills that will be installed as a family.
- Structure section lists `desktop/` and `claude-code/` with a one-line purpose each, and points at `docs/plans/` for design and implementation history.

**Step 2:** Commit: `docs: update README for desktop/claude-code split`

### Task 17: Update root CLAUDE.md for new layout

**Files:**
- Modify: `CLAUDE.md`

**Step 1:** Rewrite to reflect the new structure.

- Remove the "Three copies of SKILL.md must stay in sync" section — it now applies only inside `desktop/`.
- Add a "Desktop subtree" section with the existing three-copies rule and the repackaging commands, now pointing at `desktop/academic-essay-framework/SKILL.md`, `desktop/academic-essay-framework-SKILL.md`, and `desktop/academic-essay-framework.skill`.
- Add a "Claude Code subtree" section: 10 skills, flat; sub-skills never name each other; orchestrator owns the chain; source of truth for framework rules is split across sub-skills but all traceable to `desktop/academic-essay-framework/SKILL.md` as the original.
- Add install convention: `cp -r claude-code/academic-essay* ~/.claude/skills/`.
- Add testing convention: `claude-code/TESTING.md` is the manual-run checklist.

**Step 2:** Commit: `docs: update CLAUDE.md for desktop/claude-code split`

---

## Phase 8: Dog-food and merge

### Task 18: Dog-food end-to-end on a real essay

**Files:** none (validation only)

**Step 1:** Install the claude-code skills user-wide.
```bash
cp -r claude-code/academic-essay* ~/.claude/skills/
ls ~/.claude/skills/ | grep academic-essay | wc -l   # expect 10
```

**Step 2:** In a fresh Claude Code session, pick a past source-analysis essay (preferably one with a known grade) or a current live assignment. Create a workspace directory, run `academic-essay` from scratch, walk the full chain interactively. Capture observations in `claude-code/TESTING.md` under a "Dog-food log" subsection: dated, which essay, which stages activated correctly, where the skill failed to trigger or produced wrong output.

**Step 3:** Repeat in autoplan mode on a different assignment (or the same one, reset). Verify the taste gate surfaces at expected points and the final-gate log is usable.

**Step 4:** For any skill that failed to trigger, return to its task and tighten the `description`. Rerun that skill's activation smoke test. Commit: `fix(claude-code): tighten <skill> description after dog-food`.

**Step 5:** For any skill that produced clearly wrong output (e.g., framework-check missed a gap, style-check missed a banned phrase), return to its task and extend the body. Commit: `fix(claude-code): <skill> body corrections after dog-food`.

**Step 6:** Update `TESTING.md` Dog-food log with final observations.
```bash
git add claude-code/TESTING.md
git commit -m "docs(testing): record dog-food session results"
```

### Task 19: Merge `claude-code-upgrade` branch to `main`

**Files:** none (git only)

**Step 1:** Confirm all tasks above are committed and pushed to origin on the feature branch.
```bash
git status   # expect clean
git log --oneline main..claude-code-upgrade  # review the full commit series
git push origin claude-code-upgrade
```

**Step 2:** Ask the user: merge to `main` directly, or open a PR for review? Default recommendation: open a PR so the full commit series has a reviewable diff on GitHub.

**Step 3:** If direct merge:
```bash
git checkout main
git merge --no-ff claude-code-upgrade -m "Merge claude-code-upgrade: split Desktop and Claude Code distributions"
git push origin main
```

**Step 3 alt:** If PR:
```bash
gh pr create --title "Split Desktop and Claude Code distributions; build 10-skill workflow" --body "$(cat <<'EOF'
## Summary
- Moves Desktop skill to desktop/ (unchanged content)
- Adds claude-code/ with a 10-skill workflow: orchestrator + 9 stage skills
- Orchestrator has decision table, mid-flow resume/redo/skip, and autoplan mode with taste-decision gate
- Adds claude-code/TESTING.md with activation prompts and manual test paths

Design: docs/plans/2026-04-17-claude-code-upgrade-design.md
Plan: docs/plans/2026-04-17-claude-code-upgrade-implementation.md

## Test plan
- [ ] All 10 skills activate on their TESTING.md trigger prompts in a fresh Claude Code session
- [ ] Happy-path interactive walk produces all 10 output files
- [ ] Autoplan produces autoplan-log.md and pauses at taste-gate conditions
- [ ] Loop-back re-runs quote-verify when draft.md adds new quotes
- [ ] Mid-flow resume works across sessions
- [ ] Export produces a properly formatted .docx (Times New Roman 12pt, double-spaced, page numbers)
- [ ] Desktop .skill bundle still installs and works unchanged

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

---

## Deferred decisions

The following are intentionally NOT in this plan and should be handled separately after the initial build lands:

1. Whether to add slash-command entry points (`/essay`, `/essay-auto`) in addition to the skill — revisit after dog-food confirms the skill-only path is usable.
2. Fate of `desktop/academic-essay-framework-repo.zip` (the manually-created snapshot) — regenerate or delete once the repo structure is stable.
3. Whether to register the claude-code skills as a Claude Code plugin for namespaced install (`academic-essay:<skill>`) instead of flat — defer until there's evidence flat naming collides with other skills.

---

## Skills referenced

- @superpowers:executing-plans — use to implement this plan task-by-task
- @superpowers:subagent-driven-development — use if executing in the current session with fresh subagents per task
- @superpowers:verification-before-completion — use before claiming any task complete
