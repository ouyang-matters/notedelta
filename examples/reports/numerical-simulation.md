### ❌ FAIL NoteDelta: candidate.ipynb

**1 regression, 2 numerical changes** - `baseline.ipynb` → `candidate.ipynb`

Cells +0 / −0 / modified 6 / moved 0 · outputs changed 5 · numbers compared 14 · reproducibility warnings 1

**Failed:** 1 metric regression(s): reconstruction_error

| Metric | Baseline | Candidate | Difference | Tolerance | Status |
|---|---:|---:|---:|---|---|
| Relative reconstruction error | 0.0073 | 0.0091 | +0.0018 (+24.66%) | ±0.0001 or ±2% | ▼ Regression |
| Total energy | 12.48331 | 12.48329 | -0.00002 (-0.00016%) | ±0.001% | ≈ Within tolerance |
| Energy drift | 1.20e-06 | 1.21e-06 | +1e-8 (+0.83%) | ±5% | ≈ Within tolerance |
- **Relative reconstruction error**: Relative reconstruction error increased by 0.0018 (+24.66%) (0.0073 → 0.0091), exceeding the configured tolerance (absolute 0.0001 or relative 2% of \|baseline\|; allowed change 0.000146). The metric is declared 'minimize' (lower is better), so this is a regression.

<details><summary>Discovered numerical changes (2)</summary>

| Cell | Value | Baseline | Candidate | Difference | Status |
|---|---|---:|---:|---:|---|
| #3 | Max residual | 3.2e-08 | 4.1e-08 | +9e-9 (+28.12%) | Changed |
| #6 | Out\[6\]: u\[N // 2, ::16\] | array (8,) | array (8,) | max \|Δ\| 4.843e-05; 7/8 changed | Changed |

</details>

<details><summary>Environment & reproducibility (1)</summary>

- *warning* - No dependency declaration or lockfile was found for the candidate notebook; the package environment cannot be described or reproduced from the project files.

</details>

<sub>NoteDelta 0.1.0 · facts are observations, hypotheses are unverified. Scientific interpretation remains yours.</sub>
