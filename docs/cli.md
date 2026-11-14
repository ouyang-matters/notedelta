# CLI reference

```
notedelta [--version] COMMAND ...
```

| Command | Purpose | Runs notebook code? |
|---|---|---|
| `compare BASELINE CANDIDATE` | compare two notebook files | no |
| `compare-git NOTEBOOK... [--ref REF]` | compare notebooks with their version at a git revision | no |
| `check-config [FILE]` | validate a `.notedelta.yml` | no |
| `init [DIR]` | write a starter `.notedelta.yml` (refuses to overwrite without `--force`) | no |
| `schema [report\|config]` | print the JSON Schema of the report or the configuration | no |
| `execute NOTEBOOK --trust` | re-run a notebook into copies (opt-in) | **yes** |
| `stability RUN RUN...` | describe configured-metric variation across runs | no |

## Exit codes

| Code | Meaning |
|---|---|
| 0 | success; no fail-on condition met |
| 1 | the comparison succeeded and a fail-on condition was met |
| 2 | invalid input: missing/corrupt notebook, invalid configuration, bad arguments, unknown git revision |
| 3 | internal error, execution failure or timeout |

Errors are printed to stderr as `notedelta: error: ...` without a traceback. Use
`--debug` (or `NOTEDELTA_DEBUG=1`) to see tracebacks of internal errors.

## `compare`

```
notedelta compare BASELINE CANDIDATE [options]
```

Configuration

| Option | |
|---|---|
| `--config FILE` | use this configuration (default: `.notedelta.yml`/`.notedelta.yaml` searched from the candidate's folder up to the git root, then from the current folder) |
| `--no-config` | ignore configuration files |
| `--no-discovery` | report configured metrics only, not automatically discovered numbers |
| `--no-env` | skip environment and reproducibility diagnostics |
| `--baseline-project DIR`, `--candidate-project DIR` | folders whose dependency files describe each side (default: each notebook's folder, searching upwards to the repository root) |

Output

| Option | |
|---|---|
| `-f, --format {text,json,html,markdown}` | what goes to stdout (default `text`) |
| `-o, --output FILE` | write the `--format` output to FILE instead of stdout |
| `--json FILE`, `--html FILE`, `--markdown FILE` | additionally write these reports (several may be combined) |
| `--embed-images` | embed changed PNG/JPEG/GIF/WebP outputs side by side in the HTML report (≤ 2 MB each) |
| `--no-redact` | keep secrets and home-directory paths in excerpts (redaction is on by default) |
| `-v, --verbose` | list every cell change, within-tolerance change and notice |
| `-q, --quiet` | print nothing; on failure, one line on stderr |
| `--color {auto,always,never}` | terminal colours (`NO_COLOR` is honoured) |

Checks

| Option | |
|---|---|
| `--fail-on-regression` | exit 1 when a configured metric is classified as a regression |
| `--fail-on CATS` | exit 1 on any of: `regression`, `output_missing` (a required metric was not found), `incomparable`, `changed` (a metric or discovered number changed beyond tolerance), `new_error` (a cell output an error that it did not before), `reproducibility_warning`, `structural` (cells added/deleted/modified). Repeatable or comma separated. |
| `--max-notebook-mb MB` | refuse larger notebooks (default 256) |

The effective fail-on set is the union of the configuration's `policy.fail_on` and the
command-line options. NoteDelta never overwrites an input notebook and refuses to write a
report to a `.ipynb` path.

## `compare-git`

```
notedelta compare-git NOTEBOOK... [--ref REF] [--candidate-ref REF] [--report-dir DIR] [--summary FILE] [compare options]
```

Compares each notebook in the working tree (or at `--candidate-ref`) with the same file
at `--ref` (default `HEAD`). Dependency files (`requirements*.txt`, `pyproject.toml`,
lockfiles, `environment.yml`) are read from the same revisions, so dependency changes
between the two commits are reported. Arguments may be globs (`'notebooks/**/*.ipynb'`;
quote them so NoteDelta, not the shell, expands them). A notebook that does not exist at
the baseline revision is reported as new and is not an error.

- `--report-dir DIR` writes `<name>.html` and `<name>.json` per notebook.
- `--summary FILE` appends a Markdown summary (use `$GITHUB_STEP_SUMMARY` in GitHub Actions).
- The exit code is 1 if any notebook met a fail-on condition.

Only `git rev-parse`, `git ls-tree` and `git show` are run. Revision names are
validated (no option-like or range syntax).

## `execute` (opt-in, runs code)

```
notedelta execute NOTEBOOK --trust [--kernel NAME] [--timeout SECONDS] [--repeat N]
                          [--output-dir DIR] [--workdir notebook|temp] [--allow-errors] [--force]
```

Requires `pip install "notedelta[execute]"`. Without `--trust`, NoteDelta asks for
confirmation in an interactive terminal and refuses otherwise. **This runs the
notebook's code with your permissions; it is not a sandbox.**

Executed copies are written as `DIR/<name>.run<k>.ipynb` (default `notedelta-runs/`)
together with `<name>.run<k>.provenance.json` (kernel, language version, host platform,
start time, duration, status, error). The original notebook is never modified. Existing
run files are not overwritten without `--force`. `--timeout` applies per cell. With
`--workdir temp` the kernel starts in an empty temporary folder (relative data paths will
not resolve). Exit code 3 if any run failed or timed out.

## `stability`

```
notedelta stability RUN.ipynb RUN.ipynb [...] [--config FILE] [--format text|json]
```

For each configured metric: values per run, minimum, maximum, range, mean, sample
standard deviation and coefficient of variation. Purely descriptive - no significance
test is performed.

## `schema`

`notedelta schema report` prints the JSON Schema of the report (`schema_version` 1.x);
`notedelta schema config` prints the schema of `.notedelta.yml` v1, usable by editors
with YAML language-server support.
