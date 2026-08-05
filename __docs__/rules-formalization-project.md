# Architecture for source-formalization projects

These rules govern any Lake project that formalizes a piece of mathematical literature unit by unit. A *unit* is a directory holding one comparator trio—that much is fixed, since `__check__.py` pairs the three files by the directory they share—but **which slice of the source a unit corresponds to, and what its directory is called, are the project's own choice**, fixed in its `CLAUDE.md`: a chapter, a section, one theorem together with the lemmas it needs, or a single unit at the library root when the source is short enough not to want dividing. Nothing below depends on which.

That `CLAUDE.md` also fixes the project's root namespace, its source of truth (the paper, a reading note taken from it, or both), whether it keeps a `CompareMathlib.lean`, and any statement policy layered on top of these rules.

Two companion documents carry the parts not repeated here: `./rules-comparator.md` (the Challenge / Development pair—the specification-versus-proof split that these rules assume throughout) and `./rules-documentation.md` (module and declaration docstrings, and citations).

## The project is a Lake package

Each project is a self-contained Lake package with its own `lakefile.toml`, `lean-toolchain`, `lake-manifest.json`, and (gitignored, machine-local) `.lake/`, held in a repository of its own. The sources sit in the library directory named after the project:

```
<Project>/
├── lakefile.toml   lean-toolchain   lake-manifest.json   .gitignore
├── README.md   CLAUDE.md   AGENTS.md → CLAUDE.md
├── __docs__/            the project's own copies of the rules it follows
├── __check__.py         the project's own copy of the structural checker
├── <Project>.lean       the root all-import module
└── <Project>/           the library source tree
```

The `__docs__/` copies and `__check__.py` arrive with the template repository the project is generated from, and are the project's own from then on: they are fine-tuned in place to fit it, and a project keeps only the rules that apply to it, deleting the rest.

Module names mirror file paths under the package root: `<Project>/<Unit>/Foo.lean` is the module `<Project>.<Unit>.Foo`.

**The root module `<Project>.lean` directly imports every module of the project**, so a plain `lake build` cannot silently omit a new file. Keep the import list sorted, and add to it whenever a module is added or renamed. The exceptions are `Challenge.lean` and `CompareMathlib.lean`, which declare the same names in the same namespace as `Development.lean` and so cannot enter the same environment; the root module imports Development, and the other two are built by name.

## What a unit holds

Three files are named by the comparator discipline. The rest is production code, and how it is split is not prescribed.

- **`Challenge.lean` — the frozen statement of the unit's targets**, one declaration per numbered claim of the source, each proved by `sorry`, over its own clones of the definitions they mention. Reading it top to bottom should read like the source unit itself. It imports only Mathlib, and it is the file that does *not* change when a proof is found.
- **`Development.lean` — the same declarations, discharged** by delegating to the unit's proof files, each body bridging from the clone to the production original (`./rules-comparator.md`). This is the unit's public, source-facing face.
- **`CompareMathlib.lean`** — optional; see `./rules-comparator.md`.

Beside them:

- **The production definitions** the unit's targets are stated over: structures, instances, notation, and `rfl`-level unfolding lemmas. Give them a file of their own—`Defs.lean` is the conventional name, and nothing enforces it—as soon as more than one file states lemmas over them, so that each of those files can import the definitions rather than one of them owning the definitions and the rest having to import it whole. A unit whose targets are stated in Mathlib's vocabulary alone wants no such file, and clones nothing into its comparator files either.
- **The proof files.** One per goal, or per tight cluster of goals, named in UpperCamelCase after the result proved, with the source tag recorded in the module docstring. This is where the actual multi-line proofs live, and what `Development.lean` delegates to.

However the production side is split, the comparator files clone the definitions their targets need rather than importing them, so the production copy and the clones are maintained in step by hand (`./rules-comparator.md`).

## Import discipline

Lean's import graph is acyclic, and the comparator sits at the top of the unit, so nothing in a unit may import its own `Development.lean`. The flow within a unit is fixed:

```
production definitions  ←  proof files  ←  Development.lean

Mathlib  ←  Challenge.lean,  CompareMathlib.lean
```

- Proof files import wherever the production definitions live, and one another as needed, never `Development.lean`. That is the argument for giving the definitions a file of their own once there are two proof files: without it, one of them owns the definitions and every sibling has to import that file whole, dragging its proofs into scope along with them.
- **`Challenge.lean` and `CompareMathlib.lean` sit outside this graph entirely**—they import Mathlib alone and carry their own clones of the definitions their targets mention (`./rules-comparator.md`).
- When a production definition carries a proof obligation, prove the obligation in place when it is short; if it grows, split it into a prerequisite file imported *by* the one holding the definition. Never leave a `sorry` underneath a definition—a definition that does not elaborate takes everything downstream of it with it.
- A definition whose proof obligation *is* one of the source's numbered claims lives in the comparator files only (declared after the claim it depends on), so the obligation stays a visible goal; it has no production counterpart, and the proof files must not reference it.
- Later units build on earlier ones by importing their `Development`. A later unit's comparator files import nothing at all beyond Mathlib, so they re-clone whatever earlier definitions their own targets mention.

## Workflow

1. **Skeleton.** Write `Challenge.lean` from the source against Mathlib alone: the definitions its targets need, cloned into the comparator namespace, then every target as a `sorry`. Copy the clone block over to the production side under the project namespace, which is where the production tower will build on it. Copy `Challenge.lean` whole to `Development.lean` and adjust only its module docstring and imports. The unit must build at this stage—`sorry` is a warning, not an error.
2. **Fill.** Pick a `sorry` in `Development.lean`, create (or extend) the proof file for it, and prove the result there over the production definitions. A proof file may carry `sorry`s while work on it is in progress.
3. **Discharge.** Once the proof is `sorry`-free, add its `import` to `Development.lean` and replace the `sorry` with a body that bridges from the clones to the production API and delegates to it—for definitions that are `def`s, a one-liner; for cloned structures, the `obtain` / `⟨...⟩` conversion of `./rules-comparator.md`. **The statement never changes at this step, only its body**, and `Challenge.lean` is not touched at all.

If step 3 cannot be carried out without changing the statement, the statement was wrong: fix it in `Challenge.lean` first, propagate the identical edit to the other comparator files of the unit, and only then adjust the proof.

## Namespaces and naming

- Production declarations live in the project's root namespace; source-level objects get nested namespaces for dot notation. The comparator files share a namespace of their own (`./rules-comparator.md`).
- The comparator files own the public, source-facing names, in descriptive Mathlib style. Each proof file wraps its contents in a sub-namespace named after the file (`namespace <Root>.RhoEP` inside `RhoEP.lean`), so its concluding lemma can restate the target without a name clash; helpers not meant for use outside the file are `private`.
- Docstrings on source-facing declarations cite the source's numbering and page—see `./rules-documentation.md`.
- Modeling decisions (how a source object is encoded—e.g., `ℤ_{≥1}` as `ℕ+`) are recorded once, in the `## Implementation notes` of the `Challenge.lean` that introduces them, and stay consistent across units.
- When a declaration's natural name collides with the Mathlib lemma it mirrors, the Mathlib one is reachable as `_root_.<name>`; prefer a distinct descriptive name when the collision would confuse.

## File layout

Every `.lean` file starts with `import Mathlib`, then the project-local imports (each on its own line), then the module docstring:

```lean
import Mathlib
import <Project>.<Unit>.Defs

/-!
# <title>
...
-/
```

In `Challenge.lean` and `CompareMathlib.lean` the block stops at the first line: they take no project-local import at all.

**There is no file-level `set_option` block.** The suppressions it is tempting to open every file with—`warningAsError false`, `linter.style.longLine false`, `linter.style.emptyLine false`—are not used, and the set is empty: nothing stands between a file and the Mathlib linter set that `lakefile.toml` enables. The long-line linter in particular is a check the file is expected to pass, since comments are hard-wrapped at 100 columns (`./rules-comments.md`).

A `set_option` that changes *elaboration* rather than silencing a linter is a different matter and stays available—but only in the scoped form, attached to the one declaration that needs it and carrying a comment saying why:

```lean
set_option maxHeartbeats 1000000 in
-- The instance search for `IsDedekindDomain (integers L)` is the expensive step.
theorem foo : ... := ...
```

This is what Mathlib's own `linter.style.setOption` demands; an unscoped `set_option maxHeartbeats` at the top of a file is reported by it.

## Building

All of these are run from the project directory (from elsewhere, wrap the change of directory in a subshell so it does not leak into later commands):

```bash
lake build                               # the whole project
lake build <Project>.<Unit>.Development  # one unit
lake build <Project>.<Unit>.Challenge    # the comparator, separately
lake build <Project>.<Unit>.CompareMathlib   # likewise, when the project keeps one
```

A fresh checkout of a project needs `lake exe cache get` **before** the first build—otherwise Lean compiles Mathlib from source, which takes hours. Alongside the build, run the project's own structural check (`./rules-comparator.md`):

```bash
python3 __check__.py
```

The toolchain and the remaining `lake` commands are documented in the project's own `README.md`, one directory up from this one.
