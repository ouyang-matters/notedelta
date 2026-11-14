# Environment and reproducibility diagnostics

Inspection is read-only: notebooks and dependency files are parsed, never executed or
installed. Every finding has a **level**:

| Level | Meaning | Example |
|---|---|---|
| `fact` | directly observed in the inputs | "Kernel name differs between the compared notebooks." |
| `warning` | an observed condition that can threaten reproducibility | "No lockfile was found; exact package versions are not recorded." |
| `hypothesis` | a possible explanation that NoteDelta has **not** verified | "3 observed environment difference(s) may contribute to the changed results." |

NoteDelta never states that a dependency *caused* a numerical change, and never states
that an experiment is reproducible. Absence of findings only means that nothing was
observed in the available metadata.

## Sources

| Source | Findings |
|---|---|
| notebook `kernelspec` | `ENV-KERNEL` (name, display name, language differ), `ENV-KERNEL-MISSING` |
| notebook `language_info.version` | `ENV-PYTHON-VERSION` (minor vs patch), `ENV-PYTHON-VERSION-MISSING`, `ENV-LANGUAGE` |
| VS Code `metadata.vscode.interpreter.hash` | `ENV-INTERPRETER` |
| `nbformat` versions | `ENV-NBFORMAT` |
| execution counts | `ENV-EXEC-ORDER` (not increasing top to bottom), `ENV-EXEC-NOT-FRESH` (first count ≠ 1), `ENV-EXEC-PARTIAL` (some code cells never ran) |
| code cells (static `ast` scan) | `ENV-SEED-CHANGED` (seed calls differ: `np.random.seed`, `torch.manual_seed`, `random_state=`, `default_rng(…)`, …), `ENV-SEED-MISSING` (random APIs used, no seed found) |
| `requirements*.txt`, `pyproject.toml` (PEP 621, PEP 735, Poetry), `environment.yml` | `ENV-DECLARED` (constraint changed / only on one side), `ENV-REQUIRES-PYTHON`, `ENV-NO-LOCK` (with unpinned count), `ENV-NO-DEPS` |
| `uv.lock`, `poetry.lock`, `pdm.lock`, `Pipfile.lock` | `ENV-LOCK-VERSION`, `ENV-LOCK-ADDED`, `ENV-LOCK-REMOVED`, `ENV-LOCK-MISMATCH` (lock contradicts a declared pin) |
| notebook Python vs `requires-python` / conda `python=` | `ENV-PYTHON-CONSTRAINT` |
| files present on one side | `ENV-DEPFILE-ADDED`, `ENV-DEPFILE-REMOVED`; `ENV-DEPS-IDENTICAL`, `ENV-DEPS-SHARED` (neutral facts) |
| co-occurrence | `ENV-HYPOTHESIS` - added only when there are environment differences **and** result changes |

## Where dependency files are found

From each notebook's folder upwards, the nearest file of each kind is used, stopping at
a folder that contains `.git`, `.notedelta.yml` or `.hg` (at most six levels). Override
with `--baseline-project` / `--candidate-project`. `compare-git` reads the files from the
compared revisions. When both notebooks resolve to the same files on disk (two notebooks
in one checkout), NoteDelta says that no dependency difference can be observed from files.

Included requirement files (`-r other.txt`) are not followed (noted). Conda match specs
such as `python=3.11` are interpreted as version prefixes. `conda-lock.yml` and
`pixi.lock` are detected but not parsed in this version.

## Seeds: scope of the static check

The check recognises common APIs called directly in notebook cells. A seed set inside an
imported helper module, a configuration file or an environment variable is invisible to
it; the warning says so. Cells starting with a cell magic (`%%bash`, …) are skipped,
line magics and shell escapes are blanked before parsing.
