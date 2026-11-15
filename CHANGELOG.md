# Changelog

All notable changes are listed here. Versions follow semantic versioning; the JSON
report and the configuration carry their own schema versions.

## 0.1.0 - 2026-11-15 (first beta)

- Renamed from the working name LabTrace to NoteDelta before the first release (the
  `labtrace` name was taken on PyPI). Package, command, configuration file
  (`.notedelta.yml`) and VS Code identifiers all use `notedelta`.

Core and CLI
- `notedelta compare`: cell matching by stable ids, identical content and similarity,
  with reorder detection and explicit ambiguity flags; source, output, error, execution
  count and metadata diffs; image outputs compared by hash.
- Numerical comparison with exact decimal arithmetic, absolute/relative tolerances,
  careful handling of zero baselines, negative values, NaN, infinities and display
  precision; arrays (shape check, elementwise, max/mean/RMS/relative L2) and pandas
  tables.
- Automatic discovery of numbers in logs, printed values, NumPy arrays, JSON outputs
  and tables, via template-aligned text and a safe literal parser.
- `.notedelta.yml` v1: configured metrics (regex, key/value, JSON path extraction),
  directions, units, tolerances, cell selectors, discovery and policy settings; strict
  validation and regex safety checks.
- Explainable classification (regression, improvement, change, within tolerance,
  missing, incomparable).
- Environment diagnostics: kernel, Python version, interpreter, execution order, seeds,
  requirements/pyproject/uv/poetry/Pipfile/conda files, graded as fact/warning/hypothesis.
- Reports: terminal, JSON (schema 1.0, JSON Schema via `notedelta schema`), Markdown,
  self-contained script-free HTML with light/dark themes. Best-effort redaction.
- `notedelta compare-git`, `check-config`, `init`, `schema`.
- Opt-in `notedelta execute` (nbclient; timeouts, repeats, provenance; not sandboxed) and
  `notedelta stability`.

Integrations
- VS Code extension (preview): compare with git baseline or file, results view,
  report panel, jump to cell, regression/reproducibility quick picks, starter config.
- GitHub composite action with job summary, report artifact and least-privilege example
  workflow.
