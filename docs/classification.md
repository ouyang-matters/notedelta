# Classification rules

Every numerical result carries a `status`, the `rule` that produced it and an
`explanation` in plain language. The rules are explicit and deterministic; nothing is
inferred statistically from a single pair of values.

## Statuses

| Status | Meaning |
|---|---|
| `unchanged` | the values are equal (not listed in reports) |
| `within_tolerance` | the values differ by no more than the allowed difference |
| `changed` | a numerical change beyond tolerance, with no declared direction |
| `improvement` | a configured metric moved in its declared good direction beyond tolerance |
| `regression` | a configured metric moved in its declared bad direction beyond tolerance |
| `output_missing` | a value is present on one side only |
| `incomparable` | the values cannot be meaningfully compared (shape mismatch, `...` elision, ambiguous extraction) |

Structural changes (cells added/deleted/modified/moved, outputs added/removed, new
errors) and environment findings are reported separately from these statuses.

## Rules

| Rule | Applies to | Result | Condition |
|---|---|---|---|
| `R-EQUAL` | all | unchanged | exact equality of the printed values (NaN = NaN) |
| `R-TOLERANCE` | all | within_tolerance | `|c − b| ≤ max(absolute, relative × |b|)` |
| `R-DIRECTION` | configured metrics with `minimize`/`maximize` | regression / improvement | beyond tolerance; worse or better per the declared direction |
| `R-CHANGED` | configured metrics with `direction: none`, all discovered numbers, arrays, tables | changed | beyond tolerance (arrays/tables: any element beyond tolerance) |
| `R-NONFINITE` | configured metrics with a direction | regression | a finite baseline became NaN or ±∞ |
| `R-NONFINITE-CHANGE` | all others | changed | a non-finite value is involved and the values differ |
| `R-MISSING` | metrics, structured values | output_missing | found on one side only |
| `R-EXTRACTION` | metrics | incomparable | the rule matched a non-number, `occurrence: only` saw several values, or the index was out of range |
| `R-ARRAY-INCOMPARABLE` | arrays | incomparable | shape mismatch, `...` elision, or a scalar became an array |
| `R-TABLE-INCOMPARABLE` | tables | incomparable | no common labels and a different shape |

## Example explanations

These are produced verbatim by the current version:

> Relative reconstruction error increased by 0.0018 (+24.66%) (0.0073 → 0.0091),
> exceeding the configured tolerance (absolute 0.0001 or relative 2% of |baseline|;
> allowed change 0.000146). The metric is declared 'minimize' (lower is better), so
> this is a regression.

> Validation accuracy decreased by 1.4 percentage points (91.2% → 89.8%), exceeding
> the configured tolerance (absolute 0.2 percentage points; allowed change 0.2
> percentage points). The metric is declared 'maximize' (higher is better), so this is
> a regression.

> Max residual increased by 9e-9 (+28.12%) (3.2e-08 → 4.1e-08), exceeding the discovery
> tolerance (absolute 1e-12 or relative 1e-07% of |baseline|; allowed change 1e-12).
> Automatically discovered numbers have no declared direction, so this is a numerical
> change, not necessarily a regression.

## What is deliberately not done

- No significance testing on two values. Use `notedelta execute --repeat` and
  `notedelta stability` to observe run-to-run variation, and choose tolerances from that.
- No guessing of directions for discovered numbers, even when their label looks like
  "loss" or "accuracy". Declare the metric instead.
- No interpretation of images; they are compared by content hash.
