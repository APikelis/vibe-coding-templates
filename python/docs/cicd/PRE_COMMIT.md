# Pre-commit Hooks Setup Guide

## Overview

Pre-commit hooks check your code before it leaves your machine. The template is one file, [`.pre-commit-config.yaml`](../../templates/config/.pre-commit-config.yaml). It is a deliberately small set: fast checks on commit, slower checks on push, and CI as the backstop.

## Quick Start

```bash
# 1. Copy the template
cp templates/config/.pre-commit-config.yaml .pre-commit-config.yaml

# 2. Add pre-commit to the dev group (also done by the pyproject.toml template)
uv add --dev pre-commit

# 3. Install both hook types
uv run pre-commit install --install-hooks
uv run pre-commit install --hook-type pre-push

# 4. Run everything once
uv run pre-commit run --all-files
```

`make install` runs steps 3 and the dependency sync for you.

## What the Template Runs

| Stage | Hook | Source | Purpose |
|-------|------|--------|---------|
| commit | whitespace, EOF, YAML/TOML/JSON syntax, large files, case and merge conflicts, private keys, line endings | `pre-commit-hooks` | Hygiene |
| commit | `ruff-check` (`--fix`) | `ruff-pre-commit` | Lint, import sorting, security rules |
| commit | `ruff-format` | `ruff-pre-commit` | Formatting |
| commit | `gitleaks` | `gitleaks` | Secrets in staged changes |
| commit | `mypy` | local, via `uv run` | Strict type check of `src/` |
| pre-push | `pytest` | local, via `uv run` | Tests with the coverage floor from `pyproject.toml` |

### Why Ruff Alone

Ruff replaces four separate tools. Do not add them back alongside it, because they fight over formatting:

| Old tool | Replaced by |
|----------|-------------|
| black | `ruff format` |
| isort | Ruff rule set `I` |
| flake8 and plugins | Ruff rule sets `E`, `W`, `F`, `B`, `C4`, `SIM`, `UP` |
| bandit | Ruff rule set `S` |

The rule selection lives in `pyproject.toml` (`[tool.ruff.lint]`), not in the hook, so the hook, `make lint` and CI always agree.

### Why mypy and pytest Are Local Hooks

They use `language: system` with `uv run`, so they run against the project's locked environment. The alternative, a pre-commit-managed mypy mirror, installs a second copy of mypy and your dependencies' type stubs get out of sync with it.

## Pinning and Updating

Every `rev` is a full commit SHA with the tag in a comment:

```yaml
rev: c0f278f1254ccc8fb09676bcdb8d847e7db493e5 # frozen: v0.17.4
```

A tag can be moved, and a moved or compromised tag would silently change what runs on every developer machine. A SHA cannot. Update all hooks and keep the pins with:

```bash
uv run pre-commit autoupdate --freeze
```

Review the diff before committing it.

## Hook Stages

- **commit** (default): everything that finishes in a few seconds.
- **pre-push**: tests. Set with `stages: [pre-push]`. The old name `push` is deprecated.
- **manual**: run on demand with `uv run pre-commit run --hook-stage manual`.

## Common Tasks

```bash
# Run one hook on everything
uv run pre-commit run ruff-check --all-files

# Run the pre-push hooks now
uv run pre-commit run --hook-stage pre-push --all-files

# Reinstall after a broken install
uv run pre-commit clean && uv run pre-commit install --install-hooks
```

### Adding a Hook

Add a new `repo` entry with a frozen SHA, or a `local` hook that goes through `uv run`:

```yaml
- repo: local
  hooks:
    - id: my-check
      name: my check
      entry: uv run my-tool
      language: system
      pass_filenames: false
```

Keep commit-stage hooks fast. Put anything slow at `pre-push`.

### Skipping Hooks (Emergency Only)

```bash
SKIP=pytest git push        # skip one hook
git commit --no-verify      # skip all commit hooks
```

CI runs the same checks, so a skipped hook only delays the failure.

## Relationship to CI

Hooks are a convenience; CI is the control. `ci.yml` runs the same ruff and mypy commands, plus checks that hooks cannot do well:

- gitleaks over **full git history** (the hook only sees staged changes)
- `pip-audit` over the locked dependencies
- the test matrix across Python versions and operating systems

Run the CI equivalents locally with `make check`.

## Troubleshooting

**Hook not found or stale environment**
```bash
uv run pre-commit clean
uv run pre-commit install --install-hooks
```

**`uv: command not found` in a hook**
The local hooks need `uv` on `PATH` in the shell git uses. Install uv, or launch git from a shell where it is available.

**gitleaks false positive**
Add the path or rule to a `.gitleaks.toml` allowlist rather than skipping the hook.

**mypy passes in the hook but fails in CI (or the reverse)**
Both read `[tool.mypy]` from `pyproject.toml` and run `mypy src/`. If they differ, check that `uv.lock` is committed and current (`make lock-check`).

## Related Documentation

- [Pre-commit documentation](https://pre-commit.com/)
- [GitHub Actions Setup](./GITHUB_ACTIONS.md)
- [Code Quality](../testing/CODE_QUALITY.md)
- [Package Management](../PACKAGE_MANAGEMENT.md)
- [Test Coverage](../testing/TEST_COVERAGE.md)

## Template Information

- **Source**: [vibe-coding-templates](https://github.com/chrishayuk/vibe-coding-templates/blob/main/python/docs/cicd/PRE_COMMIT.md)
- **Template**: [`templates/config/.pre-commit-config.yaml`](../../templates/config/.pre-commit-config.yaml)
- **Author**: chrishayuk
