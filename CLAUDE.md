# CLAUDE.md

This file provides guidance to coding agents—Claude Code (claude.ai/code) and Codex alike—when working with code in this directory. It is surfaced to Codex as `AGENTS.md` through a symlink, so keep the guidance tool-neutral.

**This is still the template's skeleton.** Every section below is to be rewritten for the project derived from it; the placeholders say what each one has to end up recording. Delete this paragraph when they are all filled in.

## Project

*What the project formalizes.* If it is bound to a source, name the work and its cite key, and **name the bibliography that key lives in**—`@./__docs__/rules-documentation.md` cites against whatever this section says, and says so nowhere else. If the project is bound to a theorem rather than to a paper, state the theorem and say that the exposition is not being followed step by step. Name the source of truth for statements—the paper, a reading note taken from it, or both—and the form a page or theorem citation takes.

## Architecture

This is a standalone Lake package, and an auto-formalization project: it follows `@./__docs__/rules-formalization-project.md` for the per-unit architecture and `@./__docs__/rules-comparator.md` for the Challenge / Development pair on top of it. *State here the project-specific parameters those documents leave open:*

- **The unit granularity**: what slice of the source a unit is, and what its directory is called—the comparator trio shares one directory, but nothing fixes its name—and so what the module names and the build targets look like.
- **The root namespace**, and the comparator namespace beside it. Whether the project declares one flat namespace or a sub-namespace per file.
- **Where the production definitions live**: which file holds them, if any file does. They are what the proof files and `Development.lean` import, and never what `Challenge.lean` imports. Say how expensive bridging from a clone is here—cheap when the layer is all `def`s, less so when it holds a `structure`.
- **Whether the project keeps a `CompareMathlib.lean`**, and if not, why there is no contrast to draw.
- **Whether Mathlib may be used without restriction**, or whether the project draws a preliminary boundary that later units must respect.
- **Any standing exemption**—an unused-variable warning kept on purpose, an unscoped `maxHeartbeats`—with the reasoning recorded in the `__docs__/` copy that governs it, not here.

Every source file must be reachable from the root all-import module `AutoFormalization.lean`; adding a `.lean` file means adding its `import` line there in the same edit. `Challenge.lean` and `CompareMathlib.lean` are the exceptions—they share `Development.lean`'s namespace and are built by name.

## Which rules this project carries

*What was kept and what was narrowed.* The template ships four documents in `__docs__/`—`rules-comparator.md`, `rules-formalization-project.md`, `rules-comments.md`, `rules-documentation.md`—plus the structural checker `__check__.py`. All four are in force here: the comparator pair is what an auto-formalization project *is*, so a project that would drop `rules-comparator.md` was generated from the wrong template and belongs on the neutral one instead. A project whose situation differs from the generic wording edits its own copy rather than having the wording carve out an exception.

The one edit almost every project makes: `rules-comments.md` closes with a *When the target is Mathlib* section, which a project not aimed at upstreaming cuts, noting the cut in one line at the top of its copy.

These copies are this project's own and are fine-tuned here, not in the template. Run the checker before declaring work done, and a build after it—including the two files `lake build` leaves out, since they are the specification and a failure to elaborate in one is otherwise invisible:

```bash
python3 __check__.py
lake build
lake build <Project>.<Unit>.Challenge
lake build <Project>.<Unit>.CompareMathlib
```

## Editing conventions for the comments

Follow the rules in @./__docs__/rules-comments.md for Markdown-styled comments across the files, and @./__docs__/rules-documentation.md for the module docstrings, the per-declaration docstrings, and the citations.
