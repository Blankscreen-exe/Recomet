# 001 — Repo foundations

**Milestone:** M0 — Foundation

**Goal:** an installable Python package with clean git hygiene. Unglamorous, but it is the first
thing a reviewer sees.

## 1.1 Create the project with uv

From the repository root:

```
uv init --package --name recomet --python 3.13
```

This generates `pyproject.toml`, `src/recomet/__init__.py`, `.python-version` and a `README.md`.
Read what it generated before changing anything.

> **Why the `src/` layout?** With the package under `src/`, tests import the *installed* package
> rather than whatever happens to be in the working directory. Packaging mistakes surface in your
> test run instead of in your users' environments. It is the layout recommended by the Python
> Packaging Authority.

## 1.2 Complete the project metadata

In `pyproject.toml`, under `[project]`:

- `description` — one clear line.
- `requires-python = ">=3.13"`.
- `license = "Apache-2.0"` and `license-files = ["LICENSE"]` — the SPDX form from PEP 639. The old
  `{text = ...}` table is deprecated.
- `authors` — your name or handle. Omit the email unless you want it public.
- `keywords` and `classifiers` — development status Alpha, Python 3.13, the FastAPI framework,
  and `Typing :: Typed`. Do **not** add a `License ::` classifier: PEP 639 deprecates them in
  favour of the `license` expression, and the build backend warns if one is present. Check every
  classifier against PyPI's official list — a typo is silently useless.
- `[project.urls]` — Homepage, Repository, Issues and Documentation, pointing at
  `https://github.com/Blankscreen-exe/Recomet`.

Create an empty `src/recomet/py.typed`. It tells type checkers the package ships type hints (PEP 561).

## 1.3 Single source of truth for the version

The version lives only in `pyproject.toml`. In `src/recomet/__init__.py`, read it at runtime with
`importlib.metadata.version("recomet")` and expose it as `__version__`.

> **Hint:** a version written in two places will eventually disagree. Also decide what should
> happen when the package is not installed.

## 1.4 License

Download the official text rather than pasting it:

```
curl -o LICENSE https://www.apache.org/licenses/LICENSE-2.0.txt
```

In PowerShell, call `curl.exe` explicitly — plain `curl` is an alias for `Invoke-WebRequest`.

## 1.5 Git hygiene files

- **`.gitignore`** — start from GitHub's Python template (github.com/github/gitignore). Make sure it
  covers `.venv/`, `.env`, `.mypy_cache/`, `.ruff_cache/`, `.pytest_cache/`, `.coverage` and `htmlcov/`.
- **`.gitattributes`** — set `* text=auto eol=lf` and mark real binary types (`*.png`, `*.jpg`, ...)
  as `binary`.

  > **Why this matters on Windows:** the Dockerfile and shell scripts run inside Linux containers.
  > A script with CRLF line endings fails there with a baffling "no such file or directory" error,
  > and diffs fill with invisible line-ending changes.

- **`.editorconfig`** — UTF-8, LF, final newline, trim trailing whitespace; 4-space indent for
  Python; 2-space for YAML, TOML, JSON and Markdown.

## 1.6 Verify and commit

```
uv sync
uv run python -c "import recomet; print(recomet.__version__)"
```

Then make two commits:

1. `docs: add architecture overview, ADR 0012 and M0 task plan` — the `docs/` folder
2. `chore: initialize recomet package` — everything else

> **Why two commits?** Reviewers read history. Small, well-described commits show discipline, and
> Conventional Commits make automated changelogs possible later.

## Acceptance criteria

- [ ] `uv sync` succeeds and produces `uv.lock`, which is committed.
- [ ] `recomet.__version__` prints `0.1.0`, and the version is not duplicated in code.
- [ ] `pyproject.toml` has the SPDX license, project URLs, classifiers and `requires-python`.
- [ ] `git ls-files --eol` shows `i/lf` for text files.
- [ ] `git status` is clean after the two commits.

## Stretch question

Why is `uv.lock` committed for an application but often left out for a library? Be ready to
answer this in a review.
