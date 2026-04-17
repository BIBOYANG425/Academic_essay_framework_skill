# Academic Essay Framework

A Claude skill that applies a rigorous 4-paragraph framework and strict writing-style rules to academic essays, tuned especially for source-analysis and argumentative papers in humanities and social-science courses (environmental justice, urban planning, history, geography, sociology).

## What the skill does

When installed in Claude, this skill activates whenever you ask for help with an essay outline, thesis, introduction, body paragraph, conclusion, or draft. It enforces a specific 4-paragraph structure, requires word-for-word verification of every quote against its source PDF, requires that scholars align tightly with the mechanics of the primary source being analyzed, bans em dashes and a list of AI-writing clichés in all output, and caps corporate-jargon words at two uses per essay.

The framework uses four paragraphs: an introduction that moves hook, bridge, thesis; two body paragraphs that each move central argument, evidence (one or two quotes), analysis tying evidence to argument to thesis; and a conclusion that restates the thesis in new language and lands on a present-day "so what." When an argument requires multiple sources, each source runs its own argument-evidence-analysis flow inside the same paragraph, up to two quotes per paragraph.

## Installation

Download the `academic-essay-framework.skill` file from this repository and install it through Claude's settings. In the Claude web or mobile app, open Settings, navigate to the Skills section, and upload the `.skill` file. Once installed, the skill will trigger automatically whenever your request resembles academic writing help, even if you do not name the framework explicitly.

## Structure

The `academic-essay-framework/` directory contains the skill source, including `SKILL.md` with the full framework specification. The `academic-essay-framework.skill` file at the repository root is the packaged, installable version.

## How the framework came together

The rules in this skill were developed through iterative work on source-analysis essays for a college course on infrastructure and environmental justice. The patterns captured here reflect what consistently produced stronger drafts: running a framework check before giving substantive feedback, verifying quotes against source PDFs before building analysis (which caught real misattributions), matching scholars to source mechanics after professor feedback that a chosen framework did not fit the source, and stripping em dashes and corporate-jargon vocabulary from the final prose.

## License

MIT. See `LICENSE` for details.
