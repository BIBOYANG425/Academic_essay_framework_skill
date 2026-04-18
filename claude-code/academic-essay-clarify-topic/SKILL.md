---
name: academic-essay-clarify-topic
description: Use when the user drops an academic essay assignment prompt, topic idea, or course brief and no essay workspace has been started yet. Asks thoughtful clarifying questions one at a time to pin down discipline, lens, primary source type, length, due date, professor expectations, and prior feedback. Trigger aggressively whenever the user shares an assignment they have not yet scoped. Produces topic.md in the workspace.
---

# academic-essay-clarify-topic

## What this skill does

This is the first stage of the academic essay workflow. When the user arrives with a fresh assignment prompt, topic idea, or course brief — and no prior essay workspace exists — this skill interviews them one question at a time until the essay is scoped. The output is a single `topic.md` file in the workspace that downstream skills (`collect-materials`, `find-sources`, `scholar-match`, `draft`) will read for discipline, lens, and constraints. Do not collect materials, find sources, or draft anything here. Scope the assignment, write `topic.md`, and stop.

## Clarifying question checklist

Walk through these seven items in order. Skip any item the user has already answered in their opening message — do not re-ask what they already told you. For each remaining item, ask exactly one question per message, and prefer multiple-choice framing whenever the space is small enough.

1. **Discipline.** What field is this paper in?
   - Example: "Is this for a history class, a sociology class, an environmental studies class, or something else?"

2. **Lens (theoretical framework or analytical angle).** How is the professor expecting you to read the material?
   - Example: "Is the lens here environmental justice, urban planning, labor history, critical race theory, something else your professor named, or is the lens open?"

3. **Primary source type and availability.** What kind of primary source is the paper built around, and do you already have access to it?
   - Example: "Is the primary source a policy document, an archival photo, a feasibility study, an interview transcript, a novel, or something else — and do you have the full text or just an excerpt?"

4. **Length.** How long does the finished essay need to be?
   - Example: "Is this a 4-page paper, a 6-to-8-page paper, a 10+ page paper, or is it word-count based?"

5. **Due date.** When is it due?
   - Example: "Is this due this week, in two weeks, end of the month, or end of the term?"

6. **Professor expectations from prior assignments.** What patterns has this professor shown on earlier work — what do they reward, what do they mark down?
   - Example: "On your previous paper for this professor, what did they praise and what did they push back on?"

7. **Prior feedback / grade on similar work.** Has the user done something structurally similar before, and how did it land?
   - Example: "Have you written a source-analysis paper for this class before? If yes, what grade did it get and what was the main comment?"

## One question per message

Ask one thing at a time. Do not stack "what's the discipline and what's the due date?" into a single question — users answer the last thing they read and the first question gets dropped. Wait for the answer, acknowledge it briefly (one line), then ask the next item on the checklist. When the answer space is small and nameable, offer 3–5 multiple-choice options plus "other" rather than an open prompt.

Acknowledge the previous answer in one line, then ask exactly one next question. Do not combine the acknowledgment with two follow-ups, and do not list "and also" questions.

## Output file: `topic.md`

When the interview is done, write a plain-markdown `topic.md` (no YAML frontmatter) in the workspace with the `# Topic` H1 and exactly these six H2 sections, in this order:

The exact file contents (strip the fence markers when writing).

```markdown
# Topic

## Assignment summary
One paragraph restating the assignment prompt in the user's own words, plus the course name if known.

## Discipline and lens
The field (e.g., environmental studies) and the theoretical lens or analytical angle (e.g., environmental justice, or "lens open — student choice").

## Primary source(s)
The primary source(s) the paper is built around, including type (policy document, archival photo, feasibility study, interview, novel, etc.) and access status (have full text / have excerpt only / need to locate).

## Length and due date
Target length (pages or word count) and the due date in ISO format (YYYY-MM-DD) where possible.

## Professor expectations
What the professor has rewarded and penalized on prior assignments in this class, as the student recalls.

## Prior feedback
Grade and main comment on the most structurally similar prior assignment, if any. "None" is an acceptable value.
```

## Termination condition

Stop asking questions and write `topic.md` as soon as either is true:

- The user says some version of "that's enough, let's move on" / "move on" / "I think we have enough" / "stop asking, write it up."
- Every one of the seven checklist items has an answer (even if the answer is "I don't know" or "lens is open" — those are valid terminal answers and should be recorded verbatim in `topic.md`).

When the user's answer to one checklist item logically answers the next (e.g., "new course, new professor" implicitly answers both #6 and #7), record both answers and skip ahead. Do not re-ask what the user has already answered by implication.

When the user's answer clearly matches a nominal value in the template (e.g., "no prior feedback" → "None", "open lens" → "Lens open"), prefer the nominal value; otherwise record the user's answer verbatim.

Once `topic.md` is written, tell the user the file is saved and that the next skill in the workflow (`academic-essay-collect-materials`) will take it from here. Do not begin collecting materials, searching for sources, or outlining the essay — those are downstream skills' jobs.
