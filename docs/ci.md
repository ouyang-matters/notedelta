# GitHub Actions

NoteDelta ships a composite GitHub Action (`ouyang-matters/notedelta`) that installs NoteDelta,
runs `notedelta compare-git` for the selected notebooks against a baseline revision,
writes a summary to the job page, uploads the HTML/JSON reports as an artifact, links
them from the summary and fails the check according to your policy.

## Example workflow

```yaml
name: Notebook results
on:
  pull_request:
    paths: ["**/*.ipynb", ".notedelta.yml", "requirements*.txt", "pyproject.toml", "uv.lock"]
permissions:
  contents: read
jobs:
  notedelta:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0        # the baseline revision must be available
      - uses: ouyang-matters/notedelta@v0.1.0
        with:
          notebooks: |
            notebooks/**/*.ipynb
          fail-on: regression,new_error
          notedelta-package: notedelta==0.1.0
```


## Inputs

| Input | Default | |
|---|---|---|
| `notebooks` | (required) | one path or glob per line, or space-separated paths on one line |
| `baseline-ref` | PR base commit, else `HEAD~1` | the explicitly selected baseline |
| `config` | search upwards | path to `.notedelta.yml` |
| `fail-on` | `regression` | added to the config's `policy.fail_on`; empty = config only |
| `python-version` | `3.12` | |
| `notedelta-package` | `notedelta` | pip requirement (version pin, wheel path or URL) |
| `report-dir` | `notedelta-reports` | |
| `upload-artifact` | `true` | |
| `artifact-name` | `notedelta-reports` | |

Outputs: `exit-code` (0/1/2/3) and `report-dir`.

## Behaviour

- The check fails with exit code 1 when a fail-on condition is met, 2 for invalid
  input (for example an unknown baseline revision - check `fetch-depth`), 3 for errors.
  Each failure produces an `::error` annotation.
- Notebooks that are new in the pull request are reported as new, not as failures.
- Only notebooks that exist in both revisions are compared. NoteDelta does not try to
  compare unrelated notebooks, and it does not promise meaningful metric comparison
  when a notebook was rewritten.
- Permissions: the action needs only `contents: read`. Artifact upload uses the
  runner's built-in token; no repository write access, no secrets, no paid APIs.

## Verification status

The action's shell steps are run unchanged by automated tests against temporary git
repositories on Linux, macOS and Windows. The tests cover recursive globs, paths with
spaces, implicit and explicit baselines, invalid revisions, exit-code mapping, and the
summary and report files. The standard `actions/setup-python` and
`actions/upload-artifact` steps are not part of those tests. The action works once
NoteDelta is available on PyPI.
