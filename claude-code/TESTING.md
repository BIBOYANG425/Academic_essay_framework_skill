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
