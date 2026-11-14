# Configuration: `.notedelta.yml` (version 1)

NoteDelta looks for `.notedelta.yml` (or `.notedelta.yaml`) in the candidate notebook's
folder and its parents up to the repository root, then in the current folder. Use
`--config FILE` to choose one explicitly and `notedelta check-config` to validate it.
`notedelta schema config` prints the JSON Schema.

Validation is strict: unknown keys are errors (they are nearly always typos). Nothing in
the configuration is executed - there are no Python expressions, only declarative rules.

## Complete example

```yaml
version: 1                       # required

metrics:
  - name: reconstruction_error   # identifier: letters, digits, _ . -
    label: Relative reconstruction error
    description: Frobenius-norm error of the rank-k reconstruction
    direction: minimize          # minimize | maximize | none (aliases: lower_is_better, higher_is_better)
    unit: null                   # free text, or the special values "fraction" / "percent"
    required: true               # report as missing when not found (default true)
    extraction:
      type: regex
      pattern: 'Relative reconstruction error\s*=\s*([0-9.eE+-]+)'
      ignore_case: false
      occurrence: last           # first | last | only | 1-based index (negative counts from the end)
      streams: [stdout, result]  # stdout, stderr, result (execute_result/display_data text)
      cells:                     # optional selector; all given conditions must hold
        tags: [metrics]
        ids: []
        source_contains: reconstruct(
    tolerance:
      absolute: 0.0001
      relative: 0.02             # 2% of |baseline|

  - name: validation_accuracy
    direction: maximize
    unit: fraction
    extraction:
      type: key_value
      key: Validation accuracy
    tolerance: {absolute: 0.002}

  - name: f1
    direction: maximize
    extraction:
      type: json_path
      path: metrics.f1           # also $.metrics.f1, runs[0].f1, ["key with spaces"]

discovery:                       # automatically discovered numbers
  enabled: true
  tolerance: {absolute: 1.0e-12, relative: 1.0e-9}
  ignore:                        # regexes matched against a number's label and its line
    - '(?i)\b(?:time|elapsed|duration|took|runtime|wall|cpu|seconds?|ms|it/s|eta|timestamp|pid|port)\b'
  max_findings: 1000

environment:
  enabled: true
  dependency_files: true
  seeds: true

policy:
  fail_on: [regression, new_error]   # see "Policy" below

ignore:
  notebook_metadata: [my_tool]       # extra top-level metadata keys to ignore
  cell_metadata: [my_cell_key]
```

## Metrics

### `direction`

- `minimize`: an increase beyond tolerance is a **regression**, a decrease an **improvement**.
- `maximize`: the opposite.
- `none` (default): a change beyond tolerance is a **change**; it is never called a regression.

### `unit`

Display only. `fraction` treats values as proportions (0.912 is shown as 91.2%) and
reports differences and absolute tolerances in **percentage points**; `percent` marks
values that are already percentages. Any other text is appended to values.

### `tolerance`

A change is within tolerance when `|candidate − baseline| ≤ max(absolute, relative × |baseline|)`.
Both default to 0 (exact). The relative bound has no effect when the baseline is 0; see
[numerics.md](numerics.md).

### Extraction rules

All rules search the outputs of code cells (never the source), in document order, and
are applied line by line. Lines longer than 10,000 characters are skipped (and counted
in the report's notes).

| `type` | Matches | Notes |
|---|---|---|
| `key_value` | `<key> = 1.5`, `<key>: 1.5`, `<key> is 1.5`, `<key> 1.5%` | whole-word key, case-insensitive by default; spaces in the key match any whitespace |
| `regex` | your pattern | exactly one capture group, or a group named `value`; Python `re` syntax |
| `json_path` | a path inside `application/json` output data, or a printed dict/list | the value at the path must be a single number |

Recognised numbers: integers, decimals, scientific notation, `nan`, `inf`, `-inf`,
`Infinity`, an optional trailing `%`.

When several matches are found, `occurrence` decides (default `last`, which suits
training logs). `only` makes several *different* values an error ("incomparable").

### Regex safety

Patterns are compiled at load time and limited to 1,000 characters. NoteDelta rejects
patterns where a repeated group contains an unbounded repetition, such as `(a+)+` or
`(\w+\s?)*`, because those can take exponential time on crafted input. Possessive
quantifiers (`a++`) and atomic groups (`(?>...)`) are accepted. This is a structural
check, not a proof; extraction is additionally bounded by the per-line length limit.

## Discovery

Discovered numbers have no direction and are compared with `discovery.tolerance`
(default: absolute 1e-12, relative 1e-9, i.e. only floating-point noise is ignored).
`ignore` patterns are matched against the label (the text before the number) and the
line's text with numbers shown as `#`; matching numbers are not reported. The defaults
skip timings (`took 3.2 s`, `elapsed`, `it/s`, ...).

## Policy

`policy.fail_on` lists conditions that make the CLI exit with status 1:

| Value | Fails when |
|---|---|
| `regression` | a configured metric is classified as a regression |
| `output_missing` | a required configured metric was not found |
| `incomparable` | a metric or discovered value could not be compared |
| `changed` | a metric without direction, or a discovered number, changed beyond tolerance |
| `new_error` | a cell now produces an error output |
| `reproducibility_warning` | any environment difference or warning was found |
| `structural` | cells were added, deleted or modified |

Command-line `--fail-on`/`--fail-on-regression` options are added to this list.

## Versioning

`version: 1` is required. Future incompatible changes will use `version: 2`; NoteDelta
will keep reading version 1 files for at least one major release.
