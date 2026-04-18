---
name: academic-essay
description: Use whenever the user asks for help with an academic essay (humanities or social-science source-analysis or argumentative papers), pastes an assignment prompt, or explicitly invokes the essay workflow. Orchestrates a 10-stage workflow by inspecting the per-essay workspace directory, picking the next stage based on which output files exist, and invoking the corresponding sub-skill. Handles mid-flow resume, stage redo, stage skip, and loop-backs when a later stage surfaces work for an earlier stage. Trigger aggressively on any academic-writing help request.
---

# academic-essay

## What this skill does

This is the orchestrator. It does not write, verify, or audit anything itself. Its only job is to inspect the current essay workspace, decide which sub-skill should run next, and invoke it. Each sub-skill writes one named output file, then returns. You re-read the workspace state and repeat. The workflow is strictly sequential: one sub-skill at a time, never in parallel.

Every essay lives in its own workspace directory. Treat that directory as the state machine: the presence, absence, and relative modification times of the output files determine what runs next.

## A. The chain

Nine sub-skills, in typical execution order. Each writes the output file named at the end.

1. `academic-essay-clarify-topic` — writes `topic.md`: the clarified topic, assignment prompt, and argumentative angle.
2. `academic-essay-collect-materials` — writes `materials.md`: a catalogue of paths to assignment materials (syllabus, PDFs, notes), not copies of them.
3. `academic-essay-find-sources` — writes `candidate-sources.md`: candidate primary and secondary sources discovered via search.
4. `academic-essay-scholar-match` — writes `scholars.md`: the tight-fit scholar or scholars whose framework matches the topic and materials.
5. `academic-essay-framework-check` — writes `framework-review.md`: structural audit of the outline or draft against the four-paragraph framework.
6. `academic-essay-quote-verify` — writes `quote-verification.md`: word-for-word verification of every quote against its cited page.
7. `academic-essay-draft` — writes `draft.md`: the full essay as prose, using only verified quotes.
8. `academic-essay-style-check` — writes `style-audit.md`: an audit for em dashes, banned phrases, and restricted-word budgets.
9. `academic-essay-export` — writes `essay.docx`: the final Word document using `humanities-template.docx`.

## B. Decision table

Evaluate the rows top-down. Stop at the first row whose condition matches; the `Next stage` column is the sub-skill to invoke.

| Condition | Next stage | Why |
| --- | --- | --- |
| `topic.md` missing | `academic-essay-clarify-topic` | starting fresh |
| `materials.md` missing | `academic-essay-collect-materials` | have a topic, need materials |
| User wants more sources OR no clear scholars yet | `academic-essay-find-sources` | materials alone aren't enough |
| `candidate-sources.md` exists AND `scholars.md` missing | `academic-essay-scholar-match` | pick tight fit |
| `outline.md` exists AND (`framework-review.md` missing OR older than `outline.md`) | `academic-essay-framework-check` | check structure |
| Outline has quotes AND (`quote-verification.md` missing OR older than outline) | `academic-essay-quote-verify` | verify before building analysis |
| Above clean, `draft.md` missing | `academic-essay-draft` | write |
| `draft.md` exists AND newer than `quote-verification.md` | `academic-essay-quote-verify` | loop-back on new quotes |
| `draft.md` exists AND (`style-audit.md` missing OR older than `draft.md`) | `academic-essay-style-check` | audit |
| `style-audit.md` clean AND `essay.docx` missing | `academic-essay-export` | ship |

## C. Mid-flow behavior

When invoked in a workspace that already has files, do not silently run the next stage. First, report the state and offer control.

- Report the current state in one short line. Example: `Workspace state: topic.md, materials.md, scholars.md — next stage framework-check`.
- Offer the user four options:
  - `resume` — run the next stage per the decision table.
  - `redo <stage>` — re-run a prior stage.
  - `restart` — archive the whole workspace and start over.
  - `skip <stage>` — skip a stage; the user takes responsibility for any downstream breakage.

### `redo <stage>`

Archive the existing output file before re-invoking the stage. Copy the file to `.archive/<stage>-<YYYYMMDDTHHMMSSZ UTC>.md` in the workspace — the same timestamp format `academic-essay-style-check` uses for its rewrite archive. Then invoke the stage. Do not delete the archived file.

### `restart`

Archive the entire workspace under `.archive/<YYYYMMDDTHHMMSSZ UTC>/` (a single timestamped directory holding every current workspace file), then proceed as if the workspace were empty. Do not delete any files outright.

### `skip <stage>`

Append one line to `.skip-log.md` in the workspace: the skipped stage name, the UTC timestamp, and the reason the user gave. Then proceed to the stage after the skipped one. If a later stage fails because a skipped stage's output is missing, surface the error and cite the `.skip-log.md` entry so the user can see exactly which skip caused the break.

## D. Invocation convention

- When the orchestrator invokes a sub-skill, it passes the workspace path as the context. The sub-skill reads and writes inside that workspace.
- Sub-skills produce their output file, then return to the orchestrator. They do not chain forward on their own.
- After each sub-skill returns, re-read the workspace state and re-run the decision table. State changes between every stage.
- Do NOT invoke two sub-skills in parallel. This workflow is strictly sequential for reasoning consistency — later stages depend on the reasoning captured in earlier outputs, not just their file existence.

## E. Scope — what the orchestrator does NOT do

- Does NOT write essay content itself. That is `academic-essay-draft`'s job.
- Does NOT verify quotes itself. That is `academic-essay-quote-verify`'s job.
- Does NOT apply style rules itself. That is `academic-essay-style-check`'s job.
- Its only job is routing and state reporting. If you find yourself doing anything else, stop and hand off to the correct sub-skill.
