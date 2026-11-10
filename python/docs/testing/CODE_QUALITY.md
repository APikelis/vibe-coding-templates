# Code Quality Guide

⚠️ **IMPORTANT**: This guide is referenced in [python/BOOTSTRAP.md](../../BOOTSTRAP.md) and should be read when setting up code quality tools for new projects.

The toolchain is deliberately small: **Ruff** for linting, formatting, import sorting and security rules, and **mypy** for types. All configuration lives in one file, [`templates/config/pyproject.toml`](../../templates/config/pyproject.toml).

## Quick Start

```bash
uv run ruff format .              # format
uv run ruff check . --fix         # lint, sort imports, apply safe fixes
uv run mypy src/                  # type check
uv run pytest --cov               # tests; coverage floor is in pyproject.toml
```

Or, with the Makefile template: `make format`, `make lint`, `make typecheck`, `make coverage`, and `make check` for everything CI runs.

There is no required order between formatter and linter: Ruff owns both, so they cannot disagree.

## Installing the Tools

Do not pin versions by hand. Let uv write the current ones:

```bash
uv add --dev pytest pytest-cov mypy ruff pre-commit
```

Dev tools belong in `[dependency-groups]` (PEP 735), which is what `uv add --dev` writes. The old `[tool.uv] dev-dependencies` table is deprecated.

```toml
[dependency-groups]
dev = ["mypy", "pre-commit", "pytest", "pytest-cov", "ruff"]
```

## Tools

### Ruff: lint, format, imports, security

Ruff replaces black, isort, flake8 (and its plugins) and bandit. Do not install those alongside it: two formatters will rewrite each other's output.

```bash
uv run ruff check .                 # report issues
uv run ruff check . --fix           # apply safe fixes
uv run ruff format .                # format
uv run ruff format --check .        # CI: fail if anything would change
uv run ruff check . --select S      # one rule family
uv run ruff rule B006               # explain a rule
```

The template selects these rule families:

| Code | Family | Replaces |
|------|--------|----------|
| `E`, `W` | pycodestyle | flake8 |
| `F` | pyflakes | flake8 |
| `I` | import sorting | isort |
| `B` | bugbear | flake8-bugbear |
| `C4` | comprehensions | flake8-comprehensions |
| `UP` | pyupgrade | pyupgrade |
| `SIM` | simplify | flake8-simplify |
| `PTH` | use pathlib | flake8-use-pathlib |
| `S` | security | bandit |
| `RUF` | Ruff-specific | |

`E501` (line length) is ignored because the formatter owns line length. `S101` (assert) is ignored under `tests/**` because assert is how pytest works.

Configuration is in `[tool.ruff]`, `[tool.ruff.lint]` and `[tool.ruff.format]` in `pyproject.toml`. Do not also keep a `ruff.toml`: Ruff would use that file and ignore the one in `pyproject.toml`.

#### Suppressing a Rule

Prefer fixing the code. When a suppression is justified, scope it tightly and say why:

```python
subprocess.run(cmd, check=True)  # noqa: S603  # cmd is built from constants, no user input
```

### mypy: static types

The template turns on `strict = true`.

```bash
uv run mypy src/
```

Configuration is `[tool.mypy]` in `pyproject.toml`; do not keep a separate `mypy.ini`. The template relaxes `disallow_untyped_defs` for `tests.*` only.

Missing third-party stubs: add the specific `types-*` package for the library you use.

```bash
uv add --dev types-requests types-pyyaml
```

⚠️ **Never** install the deprecated `types-all` package. It fails to install and breaks pre-commit.

## Pre-commit Hooks

See [PRE_COMMIT.md](../cicd/PRE_COMMIT.md). Short version:

```bash
uv run pre-commit install --install-hooks
uv run pre-commit install --hook-type pre-push
uv run pre-commit run --all-files
```

Ruff and gitleaks run on commit; mypy runs on commit through `uv run`; pytest runs on push.

## CI/CD Integration

[`ci.yml`](../../templates/cicd/workflows/ci.yml) runs the same commands as the Makefile. Its quality steps are:

```yaml
- run: uv sync --dev
- run: uv run ruff check . --output-format=github
- run: uv run ruff format --check .
- run: uv run mypy src/
```

See [GITHUB_ACTIONS.md](../cicd/GITHUB_ACTIONS.md) for the rest (tests, secret scan, dependency audit, build).

## Quality Metrics

### Coverage

The floor is set once, in `pyproject.toml`, and used everywhere:

```toml
[tool.coverage.report]
fail_under = 80
```

```bash
uv run pytest --cov --cov-report=term-missing
uv run pytest --cov --cov-report=html      # browse htmlcov/index.html
```

See [TEST_COVERAGE.md](./TEST_COVERAGE.md).

### Complexity

Enable the McCabe rule in Ruff rather than adding another tool:

```toml
[tool.ruff.lint]
extend-select = ["C90"]

[tool.ruff.lint.mccabe]
max-complexity = 10
```

### Security

- **Code**: Ruff's `S` rules cover what bandit did.
- **Secrets**: gitleaks, in pre-commit (staged changes) and CI (full history).
- **Dependencies**: `pip-audit` over the locked set, via `make audit` and the `audit` job in CI. This replaces `safety`.

## Best Practices

### Type Hints

Use built-in generics and `X | None`, not `typing.List`, `Dict` or `Optional`. Ruff's `UP` rules enforce this.

```python
from typing import Any


def process_data(
    items: list[str],
    config: dict[str, Any] | None = None,
) -> dict[str, int]:
    """Map each item to its length."""
    config = config or {}
    return {item: len(item) for item in items}
```

### Docstrings

Use Google-style docstrings:

```python
def calculate_average(numbers: list[float]) -> float:
    """Calculate the average of a list of numbers.

    Args:
        numbers: Numbers to average.

    Returns:
        The arithmetic mean.

    Raises:
        ValueError: If the list is empty.
    """
    if not numbers:
        raise ValueError("Cannot calculate average of empty list")
    return sum(numbers) / len(numbers)
```

### Error Handling

Raise specific exceptions and chain the cause (Ruff rule `B904` checks this):

```python
import json
from pathlib import Path
from typing import Any


class ConfigurationError(Exception):
    """Raised when configuration is invalid."""


def load_config(path: Path) -> dict[str, Any]:
    """Load configuration from a JSON file."""
    if not path.exists():
        raise ConfigurationError(f"Config file not found: {path}")
    try:
        data: dict[str, Any] = json.loads(path.read_text())
    except json.JSONDecodeError as e:
        raise ConfigurationError(f"Invalid JSON in {path}: {e}") from e
    return data
```

## Makefile Integration

The [Makefile template](../../templates/Makefile) already provides these targets; copy it rather than writing your own:

| Target | Does |
|--------|------|
| `make format` | `ruff format` then `ruff check --fix` |
| `make lint` | `ruff check` and `ruff format --check`, no changes |
| `make typecheck` | `mypy src` |
| `make coverage` | pytest with coverage and the floor from `pyproject.toml` |
| `make audit` | `pip-audit` on the locked dependencies |
| `make secrets` | gitleaks (needs gitleaks installed) |
| `make check` | `lock-check lint typecheck coverage`, i.e. what CI runs |

## VS Code Integration

Install the **Ruff** extension (`charliermarsh.ruff`). The Python extension's `python.linting.*` and `python.formatting.*` settings are deprecated and should not be used.

`.vscode/settings.json`:

```json
{
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  }
}
```

## Quality Gates

Enforce in CI:

1. **Coverage**: floor from `pyproject.toml` (default 80%)
2. **Types**: mypy strict passes on `src/`
3. **Lint and format**: zero Ruff findings
4. **Secrets**: none in history
5. **Dependencies**: no known vulnerabilities in `uv.lock`

## Troubleshooting

**Import errors in mypy**
Add `types-*` stubs for the specific package (see above). Never `types-all`.

**Ruff and mypy disagree with an old editor setup**
Remove leftover `[tool.black]` and `[tool.isort]` tables, and uninstall the Black and isort editor extensions. Ruff reads only `[tool.ruff]`.

**Ruff ignores my settings**
A `ruff.toml` or `.ruff.toml` in the project takes precedence over `pyproject.toml`. Keep one location.

**Formatter changes files on every run**
Two formatters are active. Check for a leftover black hook or editor extension.

## Resources

- [Ruff documentation](https://docs.astral.sh/ruff/)
- [Ruff rules](https://docs.astral.sh/ruff/rules/)
- [mypy documentation](https://mypy.readthedocs.io/)
- [pre-commit documentation](https://pre-commit.com/)
- [pip-audit](https://github.com/pypa/pip-audit)
- [gitleaks](https://github.com/gitleaks/gitleaks)
