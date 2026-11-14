# Security and privacy

## Threat model

Notebooks, their outputs and configuration files may come from untrusted sources (a
pull request, a downloaded paper's supplement). NoteDelta treats all of them as data.

| Concern | Measure |
|---|---|
| Running untrusted code | Comparison never executes notebook code, shell commands or configuration. `notedelta execute` is the only code-running command; it requires `--trust` or an interactive "yes". |
| Code in configuration | `.notedelta.yml` is declarative; YAML is loaded with `safe_load` (no object construction). |
| Regular-expression DoS | patterns are length-limited, screened for nested unbounded repetition, and applied per line with a 10,000-character line cap. |
| Resource exhaustion | size limits on notebooks, outputs, lines, numbers, arrays, tables, nesting depth and similarity computations; limits are reported, not silent. |
| HTML/script injection in reports | every notebook-derived string is escaped; notebook HTML/SVG outputs are never rendered; the report has no JavaScript and a CSP of `default-src 'none'; style-src 'unsafe-inline'; img-src data:`; embedded images are restricted to validated base64 raster data. Markdown summaries escape HTML. |
| Path handling | reports never contain absolute paths (file names and project-relative paths only); NoteDelta refuses to write a report over an input notebook or to a `.ipynb` path. |
| Secrets in outputs | excerpts and diffs are redacted best-effort: private keys, AWS/GitHub/Slack/HF/OpenAI-style tokens, JWTs, `password=`-style assignments, URL credentials, and home-directory paths. **Redaction is a safety net, not a guarantee**; review reports before publishing them. `--no-redact` disables it. |
| Git | only `rev-parse`, `ls-tree` and `show` are run, as argument lists without a shell; revision names are validated (no options, no ranges). |
| VS Code workspace trust | in untrusted workspaces the extension ignores workspace-level `notedelta.pythonPath`/`notedelta.cliPath`, so a repository cannot choose the executable. The report webview runs with scripts disabled. |
| GitHub Actions | inputs reach scripts through environment variables, never `${{ }}` interpolation; the example workflow uses `permissions: contents: read`. |

## Execution is not a sandbox

`notedelta execute` runs the notebook in a Jupyter kernel subprocess with your user
permissions. The code can read and write your files and use the network. `--workdir temp`
only changes the kernel's starting folder. A container- or VM-based isolated mode would
be a separate, explicitly documented feature; it does not exist yet.

## Privacy

- No telemetry, analytics or update checks.
- No network access during comparison. (`pip`, `npm` and GitHub Actions themselves
  download packages; NoteDelta does not.)
- No accounts or licence activation.
- Reports are written only where you ask (the VS Code extension uses its own storage
  folder and deletes its report files after 7 days).

## Reporting a vulnerability

Please report security issues privately to the maintainer (see [SECURITY.md](../SECURITY.md)) rather than
in a public issue.
