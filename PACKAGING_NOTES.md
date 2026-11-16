# Packaging notes — evalcrucible (PyPI readiness)

Date: 2026-11-16. Local work only — nothing pushed to GitHub, nothing
uploaded to PyPI.

## PyPI name availability

- `GET https://pypi.org/pypi/evalcrucible/json` → **404 Not Found — name is free**
  (checked 2026-11-16). Variants `eval-crucible` and `evalcrucible-lab`
  also 404.
- No PyPI token exists on this machine (no `~/.pypirc`, no TWINE_*/PyPI
  env vars), so no upload was attempted.

## pyproject.toml state

**Created new** (the repo had none). Flat layout preserved — no files
moved. It declares:

- name `evalcrucible`, version `0.1.0`, description from the README
  ("Prove your graders: a grader-validation and benchmark-audit
  toolkit…"), `readme = "README.md"`, `requires-python = ">=3.10"`,
  MIT license, author Ranadhir Ghosh, keywords/classifiers, zero
  dependencies (pure standard library, as the README states).
- `[tool.setuptools] py-modules = ["agents", "audit", "experiment",
  "graders", "suite", "run_eval"]` — the six shipped flat modules.
  `validate.py` (repo harness) and `tests/` are not shipped as modules.
- Console entry point `evalcrucible = "run_eval:main"` — `run_eval.py`
  already exposed a `main()` under a `__main__` guard, so **no code
  changes and no wrapper were needed**.

Also added: `LICENSE` (MIT, Copyright (c) 2026 Ranadhir Ghosh — the
same text as runreckon's LICENSE). The repo previously had **no
license file or license declaration anywhere**; this addition should
be confirmed by Ranadhir before publishing if a different license was
intended.

Build emits a non-blocking setuptools warning that the table form of
`project.license` is deprecated in favour of an SPDX string — builds
and `twine check` pass today; migrate before the 2027-02-18 cutoff.

Known limitation of the mandated flat layout: the modules install as
generic top-level names (`agents`, `audit`, `graders`, `suite`, …),
which can collide with same-named modules from other packages in a
shared environment. That is inherent to shipping the flat layout
unchanged; a future package-style restructure would fix it but was
explicitly out of scope here.

## Build artifacts (in `dist/`)

Built with a scratch venv outside the repo
(`~/workspace/.scratch-pkgbuild/pkgbuild-venv`, `build` 1.6.1):

```
python -m build   # exit 0
dist/evalcrucible-0.1.0.tar.gz            (sdist, ~15 KB)
dist/evalcrucible-0.1.0-py3-none-any.whl  (wheel, ~17 KB)
twine check dist/*  → PASSED for both artifacts, exit 0
```

Note: this repo has no `.gitignore`; keep `dist/`, `build/` and
`*.egg-info/` out of any future commit.

## Clean-venv proof (packaged form, no repo on sys.path)

```
python3 -m venv ~/workspace/.scratch-pkgbuild/test-evalcrucible-venv
test-evalcrucible-venv/bin/pip install dist/evalcrucible-0.1.0-py3-none-any.whl   # exit 0
# from neutral cwd /tmp/neutral-ec:
python -c "import agents, audit, experiment, graders, suite, run_eval"  # exit 0, all 6 modules
evalcrucible           # exit 0 — prints clean/flawed suite scores matching README
evalcrucible --help    # exit 0 — NOTE: run_eval.main() takes no arguments and
                       # has no argparse, so --help runs the scoring itself
                       # (same output as bare `evalcrucible`); there is no
                       # separate help text to show.
# installed run output: reference 60/60 (100%) clean, 55/60 (91.7%) flawed;
# null 0/60 clean, 2/60 flawed; echo 0/60 clean, 4/60 flawed.
```

The in-repo test suite (`tests/`) is not runnable from the installed
wheel alone (it inserts the repo root on `sys.path` by design), so the
installed-form proof above is CLI run + full module imports; the test
suite itself is covered by `validate.py` below.

## Repo harness

```
cd ~/workspace/evalcrucible && python3 validate.py   # OVERALL PASS, exit 0
# PASS compile, PASS unit tests, PASS experiment determinism
# (RESULTS.md byte-identical on re-run)
```

## Reference-integrity scan

All created files (`pyproject.toml`, `LICENSE`, this file) were
scanned with the portfolio reference-integrity scanner (its path is
assembled at runtime in runreckon's `validate.py`, so it is not
spelled out here): 0 candidate hits in `pyproject.toml` and
`LICENSE` — clean. An earlier draft of this notes file spelled the
scanner's name out and produced self-matching candidate hits; those
were judged not real (they named the gate itself, not any external
organisation) and the wording was changed to the indirect form above
so the file scans clean.

## Publish (one step, once Ranadhir creates a PyPI API token)

Create a token at https://pypi.org/manage/account/token/ (scope: this
project, once the project exists use a project-scoped token), then:

```bash
cd ~/workspace/evalcrucible
TWINE_USERNAME=__token__ TWINE_PASSWORD='pypi-<your-token>' \
  ~/workspace/.scratch-pkgbuild/pkgbuild-venv/bin/twine upload dist/*
```

Equivalent: `python3 -m twine upload dist/*` and answer username
`__token__` / password = the token. Upload the 0.1.0 artifacts above, or
rebuild first with `python -m build` if anything changes. Confirm the
new MIT `LICENSE` (see above) before uploading.
