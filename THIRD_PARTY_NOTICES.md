# Third-party notices

NoteDelta does not bundle third-party code. It depends on the following packages, which
pip installs from PyPI under their own licences (versions measured for NoteDelta 0.1.0):

| Package | Licence |
|---|---|
| nbformat | BSD-3-Clause |
| pydantic, pydantic-core, annotated-types, typing-inspection | MIT |
| PyYAML | MIT |
| packaging | Apache-2.0 OR BSD-2-Clause |
| jsonschema, jsonschema-specifications, referencing, rpds-py, attrs | MIT |
| fastjsonschema | BSD-3-Clause |
| traitlets, jupyter_core | BSD-3-Clause |
| platformdirs | MIT |
| typing_extensions | PSF-2.0 |

Optional `notedelta[execute]`: nbclient, ipykernel, jupyter_client, pyzmq, IPython,
psutil (BSD); tornado, asttokens (Apache-2.0); debugpy, jedi, parso (MIT); libzmq
(bundled in pyzmq wheels, MPL-2.0).

The VS Code extension contains only NoteDelta code. Its build tools (TypeScript,
@vscode/vsce, @vscode/test-electron, @types/*) are development dependencies and are not
included in the `.vsix`.
