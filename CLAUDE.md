# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single Claude skill distributed in two formats from one source: a `.skill` zip bundle for Claude Desktop (uploaded via Settings → Skills) and the bare `academic-essay-framework/` directory for Claude Code (copied or symlinked into `~/.claude/skills/` or a project's `.claude/skills/`). There is no build system, no tests, no lint. All "code" is Markdown: the skill's instructions live in `academic-essay-framework/SKILL.md`, which is the source of truth for both formats.

## Three copies of SKILL.md must stay in sync

The same file content exists in three places and all three must match byte-for-byte after any edit:

1. `academic-essay-framework/SKILL.md` — canonical source
2. `academic-essay-framework-SKILL.md` — root-level copy surfaced for GitHub preview
3. `academic-essay-framework/SKILL.md` inside `academic-essay-framework.skill` (a zip archive)

After editing the canonical file, repropagate:

```bash
cp academic-essay-framework/SKILL.md academic-essay-framework-SKILL.md
rm academic-essay-framework.skill
zip academic-essay-framework.skill academic-essay-framework/SKILL.md
```

Verify with `unzip -l academic-essay-framework.skill` (should list one file: `academic-essay-framework/SKILL.md`) and `diff academic-essay-framework/SKILL.md academic-essay-framework-SKILL.md` (should be empty).

## SKILL.md structure — what to preserve when editing

The file is YAML frontmatter (`name`, `description`) followed by Markdown sections. The `description` field is how Claude decides whether to activate the skill, so it must keep enumerating concrete triggering situations ("asks for help with an essay outline," "pastes a draft," etc.) rather than being shortened to a summary. Activation is meant to be aggressive.

The body encodes a 4-paragraph framework (intro / body 1 / body 2 / conclusion), scholar-source mechanics matching rules, a quote-verification requirement, and writing-style constraints (em dash ban, banned phrases, restricted-words clusters with a two-uses-per-essay cap). These are rules the skill enforces on users' essays; they are not style rules for this repository's own prose. Edit SKILL.md normally — em dashes here in CLAUDE.md, commit messages, and README are fine.

## Scope discipline when editing the framework

The rules in SKILL.md were derived from iterative work on real course essays (see README "How the framework came together"). Don't rewrite sections for tone or generalize away specifics. Preserve concrete examples (the Winner page-121 misquote, the slow-violence mismatch example, the Roy / Bullard / Pulido scholar-source pairings) — they are load-bearing illustrations, not filler.
