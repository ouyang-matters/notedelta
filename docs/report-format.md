# Report format (JSON, schema 1.0)

`notedelta compare ... --json report.json` writes one JSON object. The authoritative
schema is generated from the code: `notedelta schema report` (also in
[report-schema.json](report-schema.json)). Every test run validates the reports of all
demo scenarios against it.

Conventions

- `*_index` fields are **0-based** cell/output/line indices; human-facing reports show them 1-based.
- Non-finite numbers are the strings `"nan"`, `"inf"`, `"-inf"`; the file is strict JSON.
- `*_text` fields carry the value exactly as printed in the notebook.
- No absolute paths: notebook `name`s are file names (or `parent/name` when both have the same name).

Top level

| Field | |
|---|---|
| `schema_version` | `"1.0"`; additive changes bump the minor version, breaking changes the major |
| `tool` | `{name, version}` |
| `generated_at` | UTC timestamp |
| `baseline`, `candidate` | name, sha256 of the file, nbformat, cell counts, kernel, language, load warnings |
| `config` | config file name (or null), metric names, whether discovery was enabled |
| `summary` | counts: cells added/deleted/modified/moved, outputs changed, new/resolved errors, numbers compared, numerical changes, regressions, improvements, missing, incomparable, reproducibility warnings, … |
| `metrics[]` | configured metrics: values with location and excerpt, `comparison`, `status`, `rule`, `explanation`, `notes` |
| `numeric_changes[]` | discovered changes: `id`, `kind` (scalar/array/table), `label`, `representation`, `location`, `status`, `rule`, `explanation`, and `scalar`/`array`/`table` details |
| `cells[]` | changed, added, deleted, moved or ambiguously matched cells: match method/similarity/ambiguity, source diff, output changes (with image hashes), execution counts, metadata keys |
| `notebook_metadata_changes[]` | top-level metadata keys added/removed/changed (volatile keys ignored) |
| `environment[]` | `{code, level, category, message, baseline, candidate, evidence}` |
| `policy` | `{fail_on, failed, reasons}` |
| `notices[]` | limits reached, matching notes, load warnings |
| `disclaimer` | the interpretation disclaimer |

Unchanged cells are not listed (their count is implied by the summary).
