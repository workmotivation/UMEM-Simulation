# Packaging and distribution

> **Not verified here.** The build described below was written against the
> documented PyInstaller and GitHub Actions APIs, but the development
> environment for this repository is headless Linux, so the Windows build has
> **never been executed**. Treat every step as untested until you have run it
> and launched the resulting executable yourself.

---

## Ways to distribute

| Method | Audience | Requires Python |
|---|---|---|
| `pip install` from source | Researchers, developers | Yes |
| Streamlit web interface | Teaching, shared servers | Yes |
| Frozen Windows executable | Users without Python | No |

The engine, the command line, and the Streamlit interface work everywhere Python
does. Freezing is only needed for users who cannot install Python.

---

## Building the Windows executable

### Prerequisites

- Windows 10 or 11, 64-bit
- Python 3.10+ from python.org (the Microsoft Store build sometimes confuses
  PyInstaller's path handling)
- About 2 GB of free disk space

### Build

```powershell
git clone https://github.com/ORGANISATION_HERE/umem-evolutionary-workbench.git
cd umem-evolutionary-workbench

.\scripts\build_windows.ps1
```

The script creates a virtual environment, installs the desktop and build extras,
runs the non-UI tests, and invokes PyInstaller. Skip the tests with
`-SkipTests`, or start from a clean tree with `-Clean`.

Output: `dist\UMEM-Workbench\UMEM-Workbench.exe`

### Verify before distributing

A green PyInstaller run does not mean a working application. At minimum:

1. Launch the executable **on a machine without Python installed**. Frozen
   builds routinely pick up modules from the developer's environment.
2. Load each preset and run a short simulation.
3. Export a bundle and open the Excel workbook.
4. Confirm the `presets\` folder shipped alongside the executable.

---

## Why the spec file looks the way it does

**Hidden imports.** SciPy, pandas, and openpyxl import parts of themselves
lazily, by name. PyInstaller's static analysis cannot see those imports, so they
are collected explicitly. Omitting them produces a build that works from source
and fails at runtime with an import error.

**Streamlit is excluded.** It is a separate way of running the application and
would add substantial weight to a desktop build that never uses it.

**UPX is disabled.** Compressing binaries frequently triggers false positives in
Windows antivirus software.

**Console is disabled.** The desktop application is windowed. When diagnosing a
build that exits silently, temporarily set `console=True` in the spec to see the
traceback.

---

## Licensing obligations

**PySide6 is LGPL v3.** Distributing a bundled executable containing Qt carries
obligations, in particular that recipients be able to replace the LGPL
components. Review the Qt licensing terms before distributing a frozen desktop
build.

The Streamlit interface, the command line, and the engine do not depend on Qt at
all, so distributing the source package carries no such obligation.

**PyInstaller is GPL v2 with a bootloader exception** that permits distributing
differently licensed applications built with it. Using it does not impose the
GPL on this project's MIT-licensed code.

See [`THIRD_PARTY_NOTICES.md`](../THIRD_PARTY_NOTICES.md).

---

## Continuous integration

Two workflows are provided in `.github/workflows/`:

- **`tests.yml`** runs the full suite on Linux, Windows, and macOS across Python
  3.10 to 3.12, then confirms every preset runs headless through the CLI.
- **`build-windows.yml`** builds the executable on a tagged release, verifies
  the artefact exists, and attaches the ZIP to the GitHub release.

Neither workflow has been executed; they will need the usual first-run
adjustments.

---

## Publishing to PyPI

```bash
pip install build twine
python -m build
twine check dist/*
twine upload dist/*
```

Before publishing, replace the placeholders (`AUTHOR_NAME_HERE`,
`AUTHOR_EMAIL_HERE`, `ORGANISATION_HERE`, `ORCID_HERE`) in `pyproject.toml`,
`CITATION.cff`, and `LICENSE`.

---

## Reducing build size

The default build is large, mostly Qt and SciPy. If size matters:

- exclude SciPy and accept the normal-approximation fallback for confidence
  intervals (`analysis/aggregation.py` already degrades gracefully);
- ship the Streamlit interface instead, which needs no Qt;
- add further `excludes` entries to the spec for unused backends.

Measure before optimising: `--log-level=DEBUG` reports what PyInstaller
collected and why.
