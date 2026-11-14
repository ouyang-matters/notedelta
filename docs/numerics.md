# Numerical comparison

## Scalars

For a baseline value *b* and candidate value *c*:

| Quantity | Definition | Undefined when |
|---|---|---|
| difference | *c − b* | either value is NaN or infinite |
| relative change | *(c − b) / \|b\|* | *b = 0*, or a value is non-finite |
| percent change | relative change × 100 | as above |
| symmetric relative difference | *\|c − b\| / ((\|b\| + \|c\|)/2)* | both are 0 |
| allowed difference | *max(absolute, relative × \|b\|)* | |
| tolerance ratio | *\|c − b\| / allowed* | allowed = 0 |

- **Exact arithmetic.** Printed values are parsed as decimals, so `0.0091 − 0.0073`
  is exactly `0.0018` (binary floating point would give `0.0018000000000000004`), and a
  difference exactly equal to the tolerance is within it.
- **Negative baselines.** The relative change uses *|b|*, so its sign always follows
  the direction of the numeric change (−2 → −1 is +50%). A note is added.
- **Sign changes.** −1 → 1 is +200%; NoteDelta adds a note that the percentage may be
  misleading.
- **Zero baseline.** Relative and percent change are reported as undefined, never as
  infinity. A relative tolerance cannot admit any change from zero, so only the
  absolute tolerance applies (and a note says so).
- **NaN and infinity.** Two NaNs are treated as equal ("unchanged", with a note);
  equal infinities are equal. Any other change involving a non-finite value has no
  defined difference. For a configured metric with a direction, a finite baseline that
  becomes NaN or ±∞ is a regression (rule `R-NONFINITE`); otherwise it is a change.
- **Display precision.** If both values look rounded for display (they contain a
  decimal point or exponent) and their difference is within the combined rounding
  intervals (e.g. 0.0073 vs 0.0074, ±0.00005 each), a note says the underlying values
  may be equal. When the two sides are printed with different numbers of decimals, a
  note says so. These notes never change the classification.

## Arrays

Arrays come from printed NumPy reprs (`array([...], dtype=...)`, the comma-less
`[[1. 2.] [3. 4.]]` format), nested numeric lists and JSON lists.

- Shapes must match; otherwise the arrays are **incomparable** and nothing is compared.
- Arrays printed with `...` (NumPy's summarised display) are **incomparable**: the
  hidden elements are unknown.
- Elementwise, each element uses the same tolerance rule as scalars, with its baseline
  element as reference. Reported: number of changed elements, number beyond tolerance,
  maximum absolute difference and its index, the values at that index, mean absolute
  difference, RMS difference and relative L2 error *‖c − b‖₂ / ‖b‖₂* (undefined if
  ‖b‖₂ = 0). NaN in the same position on both sides counts as equal; NaN on one side is
  a mismatch beyond tolerance.

## Tables

Only pandas-rendered HTML tables (`<table class="dataframe">`) are parsed. Cells are
matched by row and column label (including MultiIndex labels joined with ` / `); if
labels are duplicated but the shape is unchanged, by position. Numeric cells are
compared like scalars; changed non-numeric cells are counted. Rows or columns present
on one side only are counted and noted. Tables truncated by pandas (`...`) are compared
on the displayed cells only, with a note.

## Where discovered numbers come from

For each pair of corresponding outputs whose content differs, NoteDelta picks the most
structured representation available on both sides:

1. `application/json` data - numbers addressed by path (`metrics.acc`, `runs[0]`).
2. pandas HTML tables.
3. Printed values in `text/plain` or stream output: scalars, dicts, lists, tuples,
   `np.float64(…)`, NumPy arrays. Parsed by a small safe parser - never `eval`.
4. Otherwise, **template-aligned text**: every line is reduced to a skeleton with
   numbers replaced by placeholders (and dates, times, hex addresses, UUIDs and hashes
   masked). Lines are aligned by skeleton with an order-preserving diff, and numbers are
   paired only on lines whose skeletons are identical. `loss = 0.52` pairs with
   `loss = 0.49`; numbers on lines whose wording changed are not paired.

Numbers glued to letters (`3.2s`, `x2`, `v1.26.4`) are not treated as numbers.
Labels come from the text before a number (`Relative reconstruction error`,
`val_loss (Epoch 3/5)`).

If the two sides use different representations, both fall back to text alignment.
Outputs whose bytes are identical are not scanned at all.

## Limits

Per output: 2,000,000 characters and 50,000 lines are scanned; lines over 10,000
characters are skipped; at most 20,000 numbers; arrays up to 2,000,000 elements; tables
up to 200,000 cells. When a limit is reached the report says so in `notices`.
Notebooks over 256 MB are refused (`--max-notebook-mb`).

## Performance

The project benchmark generates a 600-cell, 14.8 MB notebook pair in which every
output changed (training logs, 2,000-element arrays, 60×8 tables, 150 KB images, JSON).
On a 6-core Windows 11 desktop with Python 3.12 a full comparison of that worst case
took 6.6 s (median of 3; 173,520 numbers compared) and rendering JSON + HTML 0.1 s.
Typical notebooks, where most outputs are unchanged, compare in well under a second.
