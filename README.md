# AutoFormalization

A GitHub template repository for a Lean 4 + Mathlib **auto-formalization** project: the package `lake new <Project> math` produces, with the Challenge / Development comparator discipline, a set of writing conventions, and a structural checker already in position, and its Mathlib pin resolved so that `lake exe cache get` works on the first clone.

What makes a project an auto-formalization project here is the comparator pair. Each unit freezes what it is chasing in a `Challenge.lean` that stands alone against Mathlib—every target stated, none proved—and carries a `Development.lean` with the identical declaration list, solved against the project's own production modules. The frozen file is a benchmark an agent can be handed on its own terms, and the matching lists are what stop a specification from being quietly weakened to fit whatever proof turned out to be reachable. The discipline is `__docs__/rules-comparator.md`; the checker enforces the parts of it that a build does not.

A project with no machine in the loop—one where a person states a result and proves it in place—wants the sibling template https://github.com/0stellensatz/LeanTemplate instead, which is this one with the comparator machinery removed.

## Deriving a project from it

Everything down to the divider is about the template. `__rename__.py` cuts it in step 2, leaving the project's own README behind.

1. **Generate the repository**, public or private, and clone it where it belongs:

	```bash
	gh repo create <Owner>/<Project> --template 0stellensatz/AutoFormalization --private --clone
	```

	A parent repository that tracks its projects as submodules drops `--clone` and runs `git submodule add` on the new repository instead, so that the submodule checkout is the only working clone. Either way, **the checkout directory has to carry the package name**—`__check__.py` reads the project name off it.

2. **Rename the package.** A GitHub template copies files verbatim and substitutes nothing, so the package, the library directory, and the root all-import module are all still called `AutoFormalization`. One script does the whole rename and then deletes itself:

	```bash
	python3 __rename__.py <Project>
	```

	It renames the library directory and the root module, rewrites every textual occurrence of `AutoFormalization`, cuts this half of the README, and deletes itself. `__check__.py` is not rewritten and does not need to be: it takes the project name from the directory it sits in—so keep the checkout directory named after the package.

3. **Fill in `lakefile.toml`**—`description`, `keywords`, and `homepage`, which ship as placeholders. `[leanOptions]`, the Mathlib requirement, and its `rev` are already what a derived project wants; leave them alone unless the whole tree is moving to a new Mathlib.

4. **Narrow `__docs__/` to the rules that actually apply**, and record in `CLAUDE.md` what was kept and what was cut. All four documents are in force for a project derived from here—the comparator pair is what this template *is*, and a project that would drop `rules-comparator.md` was generated from the wrong template. What varies is smaller: the closing *When the target is Mathlib* section of `rules-comments.md`, which a project not headed upstream cuts, and the citation section of `rules-documentation.md`, which is worth narrowing to the one source the project reads. See *What ships* below.

5. **Write `README.md` and `CLAUDE.md`.** Both ship as skeletons with their placeholders marked. `AGENTS.md` is already a symlink to `CLAUDE.md`.

6. **Build it**, from the repository root:

	```bash
	lake exe cache get   # once, before the first build—otherwise Mathlib compiles from source
	lake build
	python3 __check__.py
	```

## What ships

- **`lakefile.toml`, `lean-toolchain`, `lake-manifest.json`, `.gitignore`**—what `lake new <Project> math` emits, on the toolchain named in `lean-toolchain` and with the Mathlib revision the manifest pins. The manifest is committed, which is what lets the first `lake exe cache get` land on prebuilt oleans instead of resolving the tag afresh and drifting off the revision the sibling projects are on.
- **`AutoFormalization.lean`**—the root all-import module, empty. Every source file added under `AutoFormalization/` gets its `import` line here in the same edit, or a plain `lake build` silently skips it. The exceptions are `Challenge.lean` and `CompareMathlib.lean`, which share `Development.lean`'s namespace and are built by name. The library directory beside it holds nothing but a `.gitkeep`, since git does not track an empty directory and `__check__.py` wants the directory to exist; delete it once there is a real source file.
- **`__check__.py`**—the structural checker, and the half of the comparator discipline that is mechanical. It is textual and needs no build, and it catches the mistakes a build does not report as errors: a file missing from the root module, a comparator file that imports more than Mathlib, a module that reaches two of a unit's three comparator files, and a comparator file that has drifted from its Challenge. That third one is the reason it exists at all—two `theorem`s of syntactically identical type merge on import without a diagnostic, so a `sorry`ed Challenge target can supersede a proved Development one and nothing in the build says so.
- **`__docs__/`**—four convention documents, all four in force for a project derived from here:
	- `rules-comparator.md`—the Challenge / Development pair: what the three files are, why a comparator file imports only Mathlib, how a Development body bridges from its cloned definitions to the production API, and what the checker verifies. This is the document the template exists for.
	- `rules-formalization-project.md`—the per-unit architecture the comparator sits on top of, with the skeleton / fill / discharge workflow. Beyond the comparator trio sharing a directory it fixes no directory scheme and no filenames, naming `Defs.lean` only as a convention.
	- `rules-comments.md`—how the prose inside a comment is written. Its closing *When the target is Mathlib* section is the conditional part: a project not aimed at upstreaming cuts it and says so in one line at the top, so the copy states only rules that are in force.
	- `rules-documentation.md`—module and declaration docstrings, and citations. The citation section is worth narrowing per project.
- **`.claude/` and `.mcp.json`**—the agent wiring, shipped with the package so a derived project inherits it instead of repeating a setup step. `.claude/settings.json` registers `.claude/hooks/__lean_check__.py` on `PostToolUse` and `Stop`: it notes when an edit lands on a `.lean` file of this project, and on the session's stop runs `__check__.py` once, blocking with the report if it fails. Running it per edit would be useless—every invariant it checks spans files and is transiently false mid-edit. The hook resolves the project from its own location, so it works in any checkout and names nothing outside the repository. `.mcp.json` registers the `lean-lsp` MCP server (`uvx lean-lsp-mcp`) at project scope, so an agent reads goal states, hovers, and diagnostics off a live `lean --server` rather than inferring them from the source; it needs nothing on the machine but `uv`.
- **`.github/workflows/build.yml`**—fetches the Mathlib cache, builds, builds any comparator files by name, and runs `__check__.py`. The three workflows `lake new` ships are deliberately not here: two of them publish releases and documentation, and the third opens automatic Mathlib-bump pull requests, which would break a tree that pins one Mathlib revision across every project.
- **`LICENSE`**—Apache-2.0, matching the Lean and Mathlib ecosystem. Replace it, or delete it, if the derived project wants something else.

Once derived, these are the project's own. They are fine-tuned in place as the project's reality demands, and an exception belongs in the project's copy, never back in the template. An edit to the template changes what the *next* project starts from; it does not reach the projects already derived, and propagating it into them is a deliberate, project-by-project act—a copy that has been fine-tuned is never overwritten wholesale.

---

<!-- TEMPLATE-README-ENDS-HERE: `__rename__.py` deletes this line and everything above it, so what follows is all the derived project keeps. -->

# AutoFormalization

*What this project formalizes, and from which source. How it is laid out, unit by unit. What is proved and what is still open.*

## Building

Mathlib is pinned in `lake-manifest.json`, and `elan` will fetch the toolchain named in `lean-toolchain`. From this directory:

```bash
lake exe cache get   # once, before the first build—otherwise Mathlib compiles from source
lake build           # everything the root all-import module reaches
python3 __check__.py
```

`Challenge.lean` and `CompareMathlib.lean` are **not** among them: they share `Development.lean`'s namespace, so the root module cannot import them and `lake build` does not compile them. Build each by name, or a specification file that fails to elaborate stays green:

```bash
lake build <Project>.<Unit>.Challenge
lake build <Project>.<Unit>.CompareMathlib
```

The conventions this project follows are its own copies, in `__docs__/`.
