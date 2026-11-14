# Known limitations (0.1.0)

Matching and comparison

- Cell matching without ids is heuristic. Ambiguous cases are flagged, but a cell that
  was rewritten beyond recognition is reported as deleted + added.
- Numbers are only paired on lines whose surrounding text is identical. If the wording
  of a log line changes (`acc` → `accuracy`), its numbers are reported as text changes,
  not numeric changes. Configure a metric for values that matter.
- Numbers glued to letters (`3.2s`, `1e-3lr`) are not extracted by discovery.
- Thousands separators (`1,234.5`) are not recognised as one number.
- pandas tables are parsed from HTML only; plain-text DataFrame output falls back to
  line alignment. Other HTML tables are not parsed.
- Arrays printed with `...` cannot be compared (reported as incomparable). Print the
  values you care about, or configure a metric.
- Complex numbers are not supported.

Interpretation

- Images are compared by content hash only. A re-rendered but visually identical plot
  is reported as changed.
- No statistics are inferred from two values. `notedelta stability` is descriptive only.
- Discovered numbers never become regressions; only configured metrics with a direction do.
- Seed detection is static and limited to well-known APIs called in the notebook itself.

Environment

- Only metadata recorded in the notebooks and dependency files near them are used;
  the actually installed packages of the kernel are unknown unless the notebook printed
  them. `conda-lock.yml` and `pixi.lock` are detected but not parsed; `-r` includes in
  requirements files are not followed.

Execution

- `notedelta execute` is not sandboxed. There is no container isolation yet.
- Executing notebooks that read data with relative paths needs `--workdir notebook` (the default).

Integrations

- The VS Code extension needs the Python CLI installed separately.
- The GitHub Action's shell steps are tested against real git repositories (locally and in
  CI); the composite action has not yet been used from another repository, and it needs
  the PyPI release to install NoteDelta.
- Neither the Python package, the VS Code extension nor the action has been published.

Platform

- CI (GitHub Actions, run 42975237389 on commit 12dc8d9) passes on Linux, macOS and
  Windows with Python 3.11, 3.12 and 3.13: lint, strict type check, all tests including
  real-kernel execution, and package build. The VS Code extension's unit, end-to-end and
  in-editor integration tests pass on Linux (CI) and Windows (local).
