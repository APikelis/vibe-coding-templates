# GitHub Actions Setup Guide

## Overview

The templates give you four workflows in [`templates/cicd/workflows/`](../../templates/cicd/workflows/). One is needed by every project; the other three are optional and each says at the top when to delete it.

| Template | Copy to | Needed when |
|----------|---------|-------------|
| [`ci.yml`](../../templates/cicd/workflows/ci.yml) | `.github/workflows/ci.yml` | Always |
| [`release.yml`](../../templates/cicd/workflows/release.yml) | `.github/workflows/release.yml` | Publishing to PyPI on a `v*` tag |
| [`docs.yml`](../../templates/cicd/workflows/docs.yml) | `.github/workflows/docs.yml` | Publishing MkDocs to GitHub Pages |
| [`benchmark.yml`](../../templates/cicd/workflows/benchmark.yml) | `.github/workflows/benchmark.yml` | You have `tests/performance/` |

For a proof of concept that does not publish anything, copy `ci.yml` only.

## Quick Start

```bash
mkdir -p .github/workflows
cp templates/cicd/workflows/ci.yml .github/workflows/ci.yml
```

Then replace `{{package_name}}` (used in the wheel smoke test) and adjust the Python matrix.

## What `ci.yml` Does

| Job | Runs | Notes |
|-----|------|-------|
| `lint` | `ruff check`, `ruff format --check`, `mypy src/` | Same commands as `make lint typecheck` |
| `test` | `pytest --cov` on Python 3.12, 3.13, 3.14 (Ubuntu), plus 3.13 on macOS and Windows | Coverage floor comes from `fail_under` in `pyproject.toml`; XML uploaded to Codecov from one cell |
| `secrets-scan` | gitleaks over full history | CLI download is checksum-verified |
| `audit` | `pip-audit` on the **locked** dependencies | Exports `uv.lock`, so it audits what you ship |
| `build` | `uv build`, then imports the wheel in a clean environment | Needs `lint` and `test`; requires the package to expose `__version__` |

Run the equivalent locally with `make check`.

### Requirements This Imposes on the Project

- **`uv.lock` must be committed.** `UV_FROZEN=1` makes CI fail if the lockfile is out of date rather than silently re-resolving. Check with `make lock-check`.
- **Coverage floor lives in `pyproject.toml`** (`[tool.coverage.report] fail_under`), not in the workflow.
- **The package needs `__version__`** or the smoke test fails. Either define it or change the smoke-test command.
- **Adjust the matrix to what you support.** The template tests the current supported Python releases and one cell each on macOS and Windows. Drop the extra OS cells if you do not ship there.

## Security Defaults in the Templates

These are on in every workflow. Keep them when customising.

- **Actions pinned to commit SHAs**, with the tag in a comment (`# v7.0.1`). A moved tag cannot change what runs.
- **`permissions: contents: read`** at workflow level. A job that needs more declares it itself and only for that job.
- **`persist-credentials: false`** on checkout, so later steps cannot reuse the token.
- **`timeout-minutes`** on every job.
- **`concurrency`** so a newer push cancels the run it supersedes.

Update pins with a tool rather than by hand, and review each bump:

```bash
# Dependabot (github-actions ecosystem) or Renovate will open PRs that keep SHA + comment in step.
```

Example `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

## Release Workflow Setup

`release.yml` uses **PyPI trusted publishing (OIDC)**. No PyPI token is stored anywhere.

One-off setup on pypi.org: add your repository, the workflow filename `release.yml`, and the environment name `pypi` as a trusted publisher. In GitHub, create an environment called `pypi` (you can require a manual approval on it).

Flow on a `v*` tag: build, smoke-test the wheel, publish to PyPI (`id-token: write` on that job only), then create a GitHub release (`contents: write` on that job only).

```bash
git tag v0.1.0 && git push origin v0.1.0
```

## Docs Workflow Setup

`docs.yml` builds MkDocs with `--strict` and deploys to GitHub Pages from `main`.

```bash
uv add --group docs mkdocs mkdocs-material "mkdocstrings[python]"
```

In the repository settings, set Pages source to **GitHub Actions**. A minimal `mkdocs.yml` is in a comment at the bottom of the template.

## Benchmark Workflow Setup

`benchmark.yml` runs `tests/performance/` with pytest-benchmark and alerts at a 120% regression.

```bash
uv add --group bench pytest-benchmark
```

Storing results pushes to `gh-pages`, so write access is confined to the `store-results` job, which only runs on `main`. Pull requests never receive a write token.

## Secrets

| Secret | Used by | Notes |
|--------|---------|-------|
| `CODECOV_TOKEN` | `ci.yml` upload step | Optional; `fail_ci_if_error` is false |
| None for PyPI | `release.yml` | Trusted publishing replaces `PYPI_API_TOKEN` |

`GITHUB_TOKEN` is provided automatically and scoped by the `permissions` blocks.

## Common Trigger Patterns

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  push:
    tags: ["v*"]
  schedule:
    - cron: "0 0 * * 0"   # weekly
  workflow_dispatch:
```

Combine the keys you need under a single `on:`; a key may appear only once.

## Using uv in Workflows

```yaml
- name: Install uv
  uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
  with:
    enable-cache: true
    python-version: ${{ matrix.python-version }}

- run: uv sync --dev
- run: uv run pytest
```

`setup-uv` provides dependency caching with `enable-cache`, so a separate `actions/cache` step is not needed.

## Status Badges

```markdown
![CI](https://github.com/{username}/{repo}/actions/workflows/ci.yml/badge.svg)
![Coverage](https://codecov.io/gh/{username}/{repo}/branch/main/graph/badge.svg)
```

## Troubleshooting

**Workflow not triggering**
- Check branch names and `paths:` filters in the trigger
- Confirm the file is in `.github/workflows/`
- Validate the YAML

**`uv.lock` out of date**
Run `uv lock` locally and commit the result. CI fails on purpose here.

**Permission errors**
Grant the extra scope on the one job that needs it:
```yaml
jobs:
  my-job:
    permissions:
      pull-requests: write
```

**Wheel smoke test fails**
Make sure `{{package_name}}` is replaced and the package defines `__version__`.

**Trusted publishing rejected**
The workflow filename and environment name on PyPI must match `release.yml` and `pypi` exactly.

## Related Documentation

- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [Pre-commit Setup](./PRE_COMMIT.md)
- [Package Management](../PACKAGE_MANAGEMENT.md)
- [Test Coverage](../testing/TEST_COVERAGE.md)

## Template Information

- **Source**: [vibe-coding-templates](https://github.com/chrishayuk/vibe-coding-templates/blob/main/python/docs/cicd/GITHUB_ACTIONS.md)
- **Templates**: [`templates/cicd/workflows/`](../../templates/cicd/workflows/)
- **Author**: chrishayuk
