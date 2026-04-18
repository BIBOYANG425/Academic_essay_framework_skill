# Academic Essay Framework

A Claude skill that applies a rigorous 4-paragraph framework and strict writing-style rules to academic essays, tuned especially for source-analysis and argumentative papers in humanities and social-science courses (environmental justice, urban planning, history, geography, sociology).

## What the skill does

When installed in Claude, this skill activates whenever you ask for help with an essay outline, thesis, introduction, body paragraph, conclusion, or draft. It enforces a specific 4-paragraph structure, requires word-for-word verification of every quote against its source PDF, requires that scholars align tightly with the mechanics of the primary source being analyzed, bans em dashes and a list of AI-writing clichés in all output, and caps corporate-jargon words at two uses per essay.

The framework uses four paragraphs: an introduction that moves hook, bridge, thesis; two body paragraphs that each move central argument, evidence (one or two quotes), analysis tying evidence to argument to thesis; and a conclusion that restates the thesis in new language and lands on a present-day "so what." When an argument requires multiple sources, each source runs its own argument-evidence-analysis flow inside the same paragraph, up to two quotes per paragraph.

## Installation

The skill ships in two formats from the same source, each in its own top-level directory. Pick the one matching how you use Claude.

### Desktop / mobile / web

Download `desktop/academic-essay-framework.skill` from this repository, open Settings, go to the Skills section, and upload the `.skill` file. Once installed, the skill triggers automatically whenever your request resembles academic writing help, even if you do not name the framework explicitly.

### Claude Code

Claude Code uses a fine-grained, multi-skill workflow: one orchestrator (`academic-essay`) plus nine stage sub-skills (clarify-topic, collect-materials, find-sources, draft, quote-verify, scholar-match, framework-check, style-check, export). Install all ten as a family — the sub-skills activate automatically when their conditions are met, but most users invoke the orchestrator and let it coordinate the stages.

```bash
# User-wide, available in every project
cp -r claude-code/academic-essay* ~/.claude/skills/

# Or scoped to a single project
cp -r claude-code/academic-essay* /path/to/project/.claude/skills/
```

Verify all ten skills installed:

```bash
ls ~/.claude/skills/ | grep academic-essay | wc -l   # expect 10
```

## Structure

- `desktop/` — monolithic Desktop distribution. Contains `academic-essay-framework.skill` (the uploadable bundle), the unpacked `academic-essay-framework/` source, and a root-level `academic-essay-framework-SKILL.md` for GitHub preview. Behavior is unchanged from the previous version of this repo.
- `claude-code/` — fine-grained Claude Code distribution. Ten skill directories (one orchestrator + nine stage sub-skills, all under `academic-essay*` prefixes) plus `TESTING.md` describing how the workflow was validated.
- `docs/plans/` — design doc and implementation plan for the Claude Code workflow split.

## How the framework came together

The rules in this skill were developed through iterative work on source-analysis essays for a college course on infrastructure and environmental justice. The patterns captured here reflect what consistently produced stronger drafts: running a framework check before giving substantive feedback, verifying quotes against source PDFs before building analysis (which caught real misattributions), matching scholars to source mechanics after professor feedback that a chosen framework did not fit the source, and stripping em dashes and corporate-jargon vocabulary from the final prose.

## License

MIT. See `LICENSE` for details.
