# Security policy

Please report vulnerabilities privately, using GitHub's **Report a vulnerability**
button on the [Security tab](https://github.com/ouyang-matters/notedelta/security) of
this repository, not in a public issue. Include a minimal notebook or configuration that
reproduces the problem. You will get an acknowledgement within a week.

In scope:
- code execution or file writes caused by comparing a notebook or loading a configuration;
- script execution in generated reports;
- leaking absolute paths or secrets beyond the documented best-effort redaction;
- denial of service that bypasses the documented limits.

See [docs/security.md](docs/security.md) for the threat model.
