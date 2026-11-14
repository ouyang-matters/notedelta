# VS Code extension

Source: `extensions/vscode`. The extension is a thin client: it runs the NoteDelta CLI
(`compare` / `compare-git` with `--json --html`), reads the JSON report and presents it
with native VS Code UI. It does not re-implement the notebook editor or the analysis.

## Features

- **Compare Notebook with Git Baseline** - notebook toolbar, Explorer context menu,
  Command Palette. Prompts for the revision (default `notedelta.gitRef`, `HEAD`).
- **Compare Notebook with…** - pick any other `.ipynb` as the baseline.
- **NoteDelta view** (activity bar): summary, scientific metrics with status icons and
  values, discovered numerical changes, environment findings (warning / fact /
  hypothesis), changed cells. Clicking an item opens the notebook and selects the cell.
- **Open NoteDelta Report** - the HTML report in a webview beside the notebook (scripts
  disabled) or in the browser.
- **Show Numerical Regressions** / **Show Reproducibility Warnings** - quick picks with
  explanations; choosing a regression jumps to its cell.
- **Configure Metrics and Tolerances** - opens `.notedelta.yml`, or creates the starter
  file after a confirmation dialog.
- Status bar item with the regression count.

Commands accept optional arguments for keybindings and automation:
`notedelta.compareWith(candidateUri, baselineUri)`, `notedelta.compareWithGit(uri, ref)`.

## Missing CLI

If no NoteDelta CLI is found (setting `notedelta.cliPath`, `notedelta.pythonPath -m
notedelta`, `notedelta` on PATH, then `py`/`python3 -m notedelta`), the extension offers to
copy `pip install notedelta` and to open the settings. Unsaved notebook changes are
detected; you can save first or compare the saved file.
