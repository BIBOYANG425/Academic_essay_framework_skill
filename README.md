# Academic Essay Framework

A Claude skill that applies a rigorous 4-paragraph framework and strict writing-style rules to academic essays, tuned especially for source-analysis and argumentative papers in humanities and social-science courses (environmental justice, urban planning, history, geography, sociology).

## What the skill does

When installed in Claude, this skill activates whenever you ask for help with an essay outline, thesis, introduction, body paragraph, conclusion, or draft. It enforces a specific 4-paragraph structure, requires word-for-word verification of every quote against its source PDF, requires that scholars align tightly with the mechanics of the primary source being analyzed, bans em dashes and a list of AI-writing clichés in all output, and caps corporate-jargon words at two uses per essay.

The framework uses four paragraphs: an introduction that moves hook, bridge, thesis; two body paragraphs that each move central argument, evidence (one or two quotes), analysis tying evidence to argument to thesis; and a conclusion that restates the thesis in new language and lands on a present-day "so what." When an argument requires multiple sources, each source runs its own argument-evidence-analysis flow inside the same paragraph, up to two quotes per paragraph.

## Installation

The skill ships in two formats from the same source, each in its own top-level directory. Pick the one matching how you use Claude.

### Desktop / mobile / web

Download `desktop/academic-essay-framework.skill` directly from the GitHub UI (click the file, then the download-raw button). Open Claude's Settings → Skills and upload the `.skill` file. Once installed, the skill triggers automatically whenever your request resembles academic writing help, even if you do not name the framework explicitly. No other setup required.

### Claude Code

Claude Code uses a fine-grained, multi-skill workflow: one orchestrator (`academic-essay`) plus nine stage sub-skills (clarify-topic, collect-materials, find-sources, scholar-match, framework-check, quote-verify, draft, style-check, export). Install all ten as a family. Sub-skills activate automatically when their conditions are met, but most users invoke the orchestrator and let it coordinate the stages.

**1. Clone the repo:**

```bash
git clone https://github.com/BIBOYANG425/Academic_essay_framework_skill.git
cd Academic_essay_framework_skill
```

**2. Copy the ten skill directories into your Claude Code skills folder.** Pick one scope:

```bash
# User-wide, available in every project
cp -r claude-code/academic-essay* ~/.claude/skills/

# Or scoped to a single project
cp -r claude-code/academic-essay* /path/to/project/.claude/skills/
```

**3. Verify all ten skills installed:**

```bash
ls ~/.claude/skills/ | grep academic-essay | wc -l   # expect 10
```

**4. Install pandoc (one-time prerequisite for the export stage).** The final stage produces a `.docx` with Times New Roman 12pt, double-spacing, and page numbers using `pandoc` and the bundled `humanities-template.docx`. Earlier stages work without pandoc, but `essay.docx` export will fail cleanly if it is missing.

```bash
# macOS
brew install pandoc

# Debian / Ubuntu
sudo apt-get install pandoc

# Fedora / RHEL
sudo dnf install pandoc

# Windows: download the installer from https://pandoc.org/installing.html
```

**5. First use.** Start a fresh Claude Code session in an empty directory for the essay (e.g., `mkdir ~/essays/my-first-essay && cd $_`). Paste your assignment prompt. The `academic-essay` orchestrator will pick it up, inspect the empty workspace, and kick off `academic-essay-clarify-topic` to interview you. Every stage writes one named file (`topic.md`, `materials.md`, `scholars.md`, `draft.md`, and so on) into the workspace directory, so you can resume mid-flow in a new session by re-invoking the orchestrator from the same folder.

If a skill does not activate when you expect it to, consult `claude-code/TESTING.md` for the known trigger prompts per skill.

## Structure

- `desktop/` — monolithic Desktop distribution. Contains `academic-essay-framework.skill` (the uploadable bundle), the unpacked `academic-essay-framework/` source, and a root-level `academic-essay-framework-SKILL.md` for GitHub preview. Behavior is unchanged from the previous version of this repo.
- `claude-code/` — fine-grained Claude Code distribution. Ten skill directories (one orchestrator + nine stage sub-skills, all under `academic-essay*` prefixes) plus `TESTING.md` describing how the workflow was validated.
- `docs/plans/` — design doc and implementation plan for the Claude Code workflow split.

## How the framework came together

The rules in this skill were developed through iterative work on source-analysis essays for a college course on infrastructure and environmental justice. The patterns captured here reflect what consistently produced stronger drafts: running a framework check before giving substantive feedback, verifying quotes against source PDFs before building analysis (which caught real misattributions), matching scholars to source mechanics after professor feedback that a chosen framework did not fit the source, and stripping em dashes and corporate-jargon vocabulary from the final prose.

## License

MIT. See `LICENSE` for details.
