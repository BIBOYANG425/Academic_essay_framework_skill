# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

One skill codebase, two distributions:

- **Desktop** — a `.skill` zip bundle uploaded via Claude Desktop's Settings → Skills. Lives under `desktop/`. A single monolithic `SKILL.md` contains the whole academic-essay framework.
- **Claude Code** — a 10-skill workflow (1 orchestrator + 9 sub-skills) installed into `~/.claude/skills/`. Lives under `claude-code/`. The framework is sliced across fine-grained skills that chain through stages of essay work.

There is no build system, no test suite, no lint. All "code" is Markdown (plus one `.docx` template under `claude-code/academic-essay-export/`).

## Desktop subtree (`desktop/`)

### Three copies of SKILL.md must stay in sync

The same Desktop `SKILL.md` content exists in three places and all three must match byte-for-byte after any edit:

1. `desktop/academic-essay-framework/SKILL.md` — canonical source
2. `desktop/academic-essay-framework-SKILL.md` — root-level preview copy (surfaced for GitHub rendering)
3. `academic-essay-framework/SKILL.md` inside `desktop/academic-essay-framework.skill` (a zip archive)

After editing the canonical file, repropagate (commands run from repo root):

```bash
cp desktop/academic-essay-framework/SKILL.md desktop/academic-essay-framework-SKILL.md
rm desktop/academic-essay-framework.skill
(cd desktop && zip academic-essay-framework.skill academic-essay-framework/SKILL.md)
```

Verify:

```bash
diff desktop/academic-essay-framework/SKILL.md desktop/academic-essay-framework-SKILL.md
unzip -l desktop/academic-essay-framework.skill
```

`diff` should produce no output. `unzip -l` should list exactly one file: `academic-essay-framework/SKILL.md`.

### SKILL.md structure — what to preserve when editing

The file is YAML frontmatter (`name`, `description`) followed by Markdown sections. The `description` field is how Claude decides whether to activate the skill, so it must keep enumerating concrete triggering situations ("asks for help with an essay outline," "pastes a draft," etc.) rather than being shortened to a summary. Activation is meant to be aggressive.

The body encodes a 4-paragraph framework (intro / body 1 / body 2 / conclusion), scholar-source mechanics matching rules, a quote-verification requirement, and writing-style constraints (em dash ban, banned phrases, restricted-words clusters with a two-uses-per-essay cap). These are rules the skill enforces on users' essays; they are not style rules for this repository's own prose. Edit `SKILL.md` normally — em dashes here in `CLAUDE.md`, commit messages, and `README` are fine.

## Claude Code subtree (`claude-code/`)

### 10 skills: 1 orchestrator + 9 sub-skills

- `academic-essay` — orchestrator. The ONLY skill that knows the chain order and routes the user through stages.
- `academic-essay-clarify-topic`
- `academic-essay-collect-materials`
- `academic-essay-find-sources`
- `academic-essay-scholar-match`
- `academic-essay-framework-check`
- `academic-essay-quote-verify`
- `academic-essay-draft`
- `academic-essay-style-check`
- `academic-essay-export`

Sub-skills **never reference each other by name**. They use workspace file existence as state (e.g., "if `sources.md` exists, proceed; if not, the orchestrator will route back"). Only the orchestrator composes the chain.

### Relationship to the Desktop canonical content

Several sub-skills lift their content from the Desktop canonical file: the framework rules (→ `academic-essay-framework-check`), scholar-source matching (→ `academic-essay-scholar-match`), quote verification (→ `academic-essay-quote-verify`), and style constraints (→ `academic-essay-style-check`) all derive from the relevant sections of `desktop/academic-essay-framework/SKILL.md`.

This is **loose coupling, not auto-sync**. When Desktop canonical content changes, the corresponding Claude Code sub-skills may need a manual update. There is no script that propagates edits — a human (or Claude) has to re-read the Desktop section and revise the matching sub-skill.

### Install convention

```bash
cp -r claude-code/academic-essay* ~/.claude/skills/
```

All 10 skills drop into `~/.claude/skills/` side-by-side. The orchestrator's `name` is `academic-essay`; sub-skills use `academic-essay-<stage>`.

### Testing convention

`claude-code/TESTING.md` is the manual-run checklist — trigger prompts and expected behavior per skill. There is no scripted test suite; verification is hands-on, matching prompts to observed activations and outputs.

## Scope discipline when editing any SKILL.md

The rules in the Desktop `SKILL.md` (and its derivatives in Claude Code sub-skills) were derived from iterative work on real course essays (see `README` → "How the framework came together"). Don't rewrite sections for tone or generalize away specifics. Preserve concrete examples — the Winner page-121 misquote, the slow-violence mismatch example, the Roy / Bullard / Pulido scholar-source pairings — they are load-bearing illustrations, not filler. This applies equally to Desktop and to every Claude Code sub-skill that carries forward those passages.
