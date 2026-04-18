---
name: academic-essay-collect-materials
description: Use after topic.md exists but before any source search or analysis happens. Prompts the user to point Claude at syllabus, assignment brief, required readings, past professor feedback, and any other course materials. Indexes the file paths in the working directory without copying. Trigger when the user is ready to gather inputs for an essay in progress, or when topic.md is present but materials.md is not. Produces materials.md cataloguing paths and brief notes per file.
---

# academic-essay-collect-materials

## What this skill does

This is the second stage of the academic essay workflow. Once `topic.md` has been written by `academic-essay-clarify-topic`, the user typically already has course materials on disk — a syllabus PDF, an assignment brief, required readings, past professor feedback, maybe a photo of handwritten lecture notes. This skill interviews the user to catalogue those existing files by path. It does **not** search the internet, and it does **not** copy files into the workspace. The output is a single `materials.md` in the workspace that downstream skills (`find-sources`, `scholar-match`, `quote-verify`) read to know where on disk the user's course materials already live.

## One material at a time

Ask the user to name each material and provide its path, one at a time. Do not stack "give me the syllabus and all the readings" into one question — users respond to the last thing they read. For each file the user names:

- Verify the path exists with `ls` or `Read` before recording it. If the path is wrong, ask for a correction; do not silently skip or guess.
- Note the file type (PDF, docx, image, txt, etc.) from the extension.
- Capture a one-line description — ask the user, or derive from the filename if they say "just use the filename."

## Category structure

Group entries under these five categories. Every category must appear in `materials.md` even if empty (record `None` under an empty category — do not delete the heading).

- **Syllabus** — the course syllabus file.
- **Assignment brief** — the specific prompt / rubric / handout for this essay.
- **Required readings** — readings the professor assigned that are in scope for this essay.
- **Optional readings** — supplementary readings the student thinks may be useful.
- **Professor feedback** — past graded work, rubrics, or written comments from this professor that shape expectations.

## Edge cases

- **Path does not exist.** Tell the user the path failed and ask them to re-paste it. Do not silently skip the file — a missing material downstream is worse than a slow prompt here.
- **No materials in a category.** Record the literal text `None` under that category heading. Keep the heading so downstream skills can see the category was considered and intentionally empty.
- **User points at a directory instead of a file.** List the directory's contents with `ls`, show them to the user, and ask which specific file they meant. Do not auto-pick, and do not record the directory path itself as a material.
- **Duplicate entry.** If the user gives the same path under two categories, ask which category is the correct one rather than recording it twice.

## Do not copy files

Record paths only. Do not copy, move, symlink, or rewrite any of the user's files into the working directory. Downstream skills open files directly from the recorded paths.

## Output file: `materials.md`

When the user signals they are done (see termination condition below), write `materials.md` in the working directory with the exact section structure shown below. The exact file contents (strip the fence markers when writing).

```markdown
# Materials

## Syllabus
- path — description

## Assignment brief
- path — description

## Required readings
- path — description
- path — description

## Optional readings
- path — description

## Professor feedback
- path — description
```

Each entry is a single bullet in the form `- absolute-or-relative-path — one-line description`. Use an em dash between path and description. If a category has no entries, write `None` as the entire body of that section instead of a bullet list.

## Termination condition

Stop asking for more materials and write `materials.md` as soon as the user says some version of "that's all" / "that's everything" / "nothing else" / "move on" / "let's continue." After writing, tell the user the file is saved and that the next skill in the workflow (`academic-essay-find-sources`) will take it from here — but only run `find-sources` if the user indicates they still need additional scholarly sources beyond what is already recorded. Do not begin searching for sources, reading the files, or drafting anything — those are downstream skills' jobs.
