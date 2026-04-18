---
name: academic-essay-export
description: Use when style-audit.md is clean and the user is ready to submit the essay. Converts draft.md to a Word document using pandoc with the bundled humanities-template.docx, producing double-spaced Times New Roman 12pt text with page numbers. Fails clearly if pandoc is missing or the template file is absent — does not silently produce unformatted output. Trigger when the user asks to export, convert to docx, or prepare for submission. Produces essay.docx in the workspace.
---

# academic-essay-export

## What this skill does

This is the final sub-skill in the academic-essay workflow. It converts the workspace's `draft.md` into `essay.docx` using `pandoc` with the bundled `humanities-template.docx` reference doc. The template forces the Normal paragraph style to Times New Roman 12pt, line-spacing 2.0 (double), and puts a `PAGE` field in the footer so Word renders page numbers on every page. The output is the binary `essay.docx` and nothing else — no audit file, no structured report. This skill does not modify `draft.md`, does not re-run any style or structure check, and does not upload or submit the file anywhere.

## Preconditions

Before doing anything, check all of the following. If any precondition fails, print the specific failure and stop — do not attempt to continue with a degraded workflow.

- `draft.md` exists in the current working directory. If missing, tell the user to run `academic-essay-draft` first and stop.
- Style check has passed, or the user has explicitly acknowledged the remaining findings in `style-audit.md`. If `style-audit.md` still shows em-dash or banned-phrase findings and the user has not said "export anyway," ask them to either fix the findings or confirm they want to ship with them, and wait for the answer.
- `pandoc` is on `PATH`.
- The bundled template `humanities-template.docx` is readable in the skill directory.

## Availability checks

Run these two checks before invoking pandoc. Both must pass. Do not fall through to a "best effort" export — silently producing an unformatted docx is worse than failing.

### pandoc on PATH

```bash
command -v pandoc
```

If this returns nothing, stop and tell the user how to install pandoc for their platform:

- macOS: `brew install pandoc`
- Debian / Ubuntu: `sudo apt-get install pandoc`
- Fedora / RHEL: `sudo dnf install pandoc`
- Windows: download the installer from https://pandoc.org/installing.html

Do not attempt to install it automatically.

### Template file present and readable

```bash
test -r "<skill-dir>/humanities-template.docx"
```

`<skill-dir>` is the directory that contains this `SKILL.md`. The bundled template ships alongside this file; if it is missing, something is wrong with the skill installation and the user should reinstall the skill. Stop and say so. Do not try to fall back to pandoc's built-in reference doc — that would silently produce single-spaced Cambria output, which is the opposite of what the user asked for.

## Overwrite protection

Before writing, check whether `essay.docx` already exists in the workspace:

```bash
test -e essay.docx
```

If it exists, ask the user whether to overwrite it. Do not overwrite silently. If the user declines, stop and leave `essay.docx` in place. This is a taste-gate condition handled by the orchestrator's autoplan contract — when the orchestrator is running this sub-skill end-to-end, it surfaces this prompt rather than auto-answering.

## Pandoc command

Use the skill's own directory as the source for the template. Substitute `<skill-dir>` with the absolute path to the directory containing this `SKILL.md` (in Claude Code this is typically `~/.claude/skills/academic-essay-export/` or the project's `.claude/skills/academic-essay-export/`).

```bash
pandoc draft.md \
  --reference-doc="<skill-dir>/humanities-template.docx" \
  -o essay.docx
```

No other flags. No filters. No metadata injection. The whole point of the reference doc is that pandoc inherits its style definitions, and adding flags like `--standalone` or `--toc` would either be no-ops or would inject unwanted structure into a humanities essay.

## Output

- `essay.docx` in the working directory. Binary Word document. Times New Roman 12pt, double-spaced, page numbers in the footer.
- No markdown audit file, no structured findings. This is the one sub-skill in the workflow whose output is binary rather than a `.md` file. Do not invent a `export-report.md` to match the other sub-skills — it would be noise.

## Scope

This skill is deliberately narrow:

- It does **not** modify `draft.md` in any way.
- It does **not** re-run `academic-essay-style-check`, `academic-essay-framework-check`, or `academic-essay-quote-verify`. Those are separate skills and must have already run (or been deliberately skipped) before the user reaches export.
- It does **not** upload `essay.docx` anywhere — not to email, Drive, Canvas, Blackboard, or any LMS. Submission is the user's job; this skill only produces the file.
- It does **not** open `essay.docx` in Word or Pages. Previewing is optional and the user can double-click the file themselves.

## Termination

After the pandoc command succeeds, print a short confirmation:

```
Exported essay.docx
  path: <absolute path to essay.docx>
  size: <size in KB, from `wc -c` or `ls -l`>
```

Then stop. Do not chain into any other skill. If the user wants another revision cycle, they will invoke `academic-essay-style-check` or `academic-essay-draft` themselves.

If pandoc exits non-zero, print its stderr verbatim and stop. Do not try to recover or retry with different flags — the user needs to see the real error so they can diagnose it.
