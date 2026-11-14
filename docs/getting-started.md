# Getting started

This tutorial uses the notebooks in `examples/` (they ship with the repository) and
takes about ten minutes.

## 1. Install

```bash
python -m pip install notedelta
notedelta --version
```

## 2. Compare two notebooks

```bash
cd examples/numerical-simulation
notedelta compare baseline.ipynb candidate.ipynb
```

The candidate changed the reconstruction rank from 12 to 10. NoteDelta prints a short
summary: a configured metric (`Relative reconstruction error`) went from 0.0073 to
0.0091 - an increase of exactly 0.0018, or 24.66% - which exceeds the configured
tolerance, and because the metric is declared `minimize` it is a **regression**.
Two other numbers changed without configuration (`Max residual`, and a printed NumPy
array); they are reported as **changes**, not regressions, because NoteDelta does not
know which direction is better for them.

Nothing was executed: NoteDelta only read the saved outputs.

## 3. Look at the full report

```bash
notedelta compare baseline.ipynb candidate.ipynb --html report.html
```

Open `report.html` in any browser. It is a single file without scripts or external
resources, so it can be attached to an email or archived with your results. The
*Detailed diagnostics* section explains every classification.

## 4. Tell NoteDelta what matters

```bash
notedelta init      # writes .notedelta.yml
notedelta check-config
```

Edit the `metrics` section. Each metric needs a name, an extraction rule and,
ideally, a direction and a tolerance:

```yaml
metrics:
  - name: validation_accuracy
    label: Validation accuracy
    direction: maximize
    unit: fraction
    extraction:
      type: key_value
      key: Validation accuracy      # matches "Validation accuracy = 0.912" or "Validation accuracy: 0.912"
    tolerance:
      absolute: 0.002               # 0.2 percentage points
```

See [configuration.md](configuration.md) for regex and JSON-path extraction, cell
selectors and policies.

## 5. Use it as a check

```bash
notedelta compare baseline.ipynb candidate.ipynb --fail-on-regression -q
echo $?   # 1 when a configured metric regressed
```

Or compare against git history without exporting files:

```bash
notedelta compare-git notebooks/analysis.ipynb --ref main
```

## 6. Look at the environment

```bash
cd ../reproducibility
notedelta compare baseline/analysis.ipynb candidate/analysis.ipynb
```

The two notebooks contain the same code and nearly identical output, but were run with
different kernels, Python 3.10 vs 3.12, and NumPy 1.26.4 vs 2.1.0 (from `uv.lock`).
NoteDelta lists these as **facts**. Because the results did not change beyond the
discovery tolerance, it does not suggest that the environment changed anything; when
results do change, it adds a single clearly-labelled **hypothesis** that the
environment *may* contribute, which you can test by re-running both versions in the
same environment.

## 7. Optional: measure run-to-run variation

If you trust a notebook, you can let NoteDelta re-run it to see how much its metrics vary
between runs (this executes the notebook's code; it is not sandboxed):

```bash
pip install "notedelta[execute]"
notedelta execute analysis.ipynb --trust --repeat 3
notedelta stability notedelta-runs/analysis.run*.ipynb
```

The observed range is a useful lower bound when choosing tolerances.
