# Package Management with uv

## Overview

This project uses `uv` as the primary package manager for Python dependencies. `uv` is a fast, reliable Python package and project manager written in Rust.

## Why uv?

- **Speed**: 10-100x faster than pip and pip-tools
- **Reliability**: Built-in resolver prevents dependency conflicts
- **Simplicity**: Single tool for package and project management
- **Compatibility**: Drop-in replacement for pip commands
- **Lock files**: Automatic dependency locking for reproducible builds

## Installation

### Install uv
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Using pip (fallback)
pip install uv

# Verify installation
uv --version
```

## Basic Usage

### Installing Dependencies

```bash
# Install project with all dependencies
uv sync

# Install including dev dependencies
uv sync --dev

# Install specific package
uv add package-name

# Install dev dependency
uv add --dev pytest-cov

# Install from requirements file
uv pip install -r requirements.txt
```

### Running Commands

```bash
# Run command in project environment
uv run python script.py

# Run pytest
uv run pytest

# Run with coverage (floor is set in pyproject.toml)
uv run pytest --cov

# Run any command
uv run make test
```

### Managing Dependencies

```bash
# Add a new dependency
uv add numpy

# Add dev dependency
uv add --dev pytest-benchmark

# Remove dependency
uv remove package-name

# Update dependencies
uv sync --upgrade

# Show installed packages
uv pip list
```

## Project Configuration

### pyproject.toml

The project uses `pyproject.toml` for dependency management. Start from [`templates/config/pyproject.toml`](../templates/config/pyproject.toml).

```toml
[project]
dependencies = [
    "pydantic>=2.0.0",
    "python-dotenv>=1.0.0",
]

# Dev tools live in a PEP 735 dependency group, not the deprecated [tool.uv] dev-dependencies.
[dependency-groups]
dev = [
    "mypy",
    "pre-commit",
    "pytest",
    "pytest-cov",
    "ruff",
]
```

Do not pin the dev tools by hand. Run `uv add --dev <package>` and uv writes the current version and updates `uv.lock`. Extra groups (for example `docs` or `bench`, used by the optional workflows) are added with `uv add --group <name> <package>`.

### Lock File

`uv.lock` ensures reproducible installations:
- Automatically generated and updated
- Should be committed to version control
- Ensures all developers use same dependency versions

## Common Commands Reference

### Development Workflow

```bash
# Initial setup
git clone <repo>
cd {{project-name}}
uv sync --dev

# Run tests
uv run pytest

# Run with coverage (floor is set in pyproject.toml)
uv run pytest --cov

# Run linting
uv run ruff check .

# Run type checking
uv run mypy src/

# Format code
uv run ruff format .
```

### Package Management

```bash
# Add production dependency
uv add fastapi

# Add dev dependency
uv add --dev pytest-mock

# Add with version constraint
uv add "pandas>=2.0.0"

# Remove package
uv remove pandas

# Update all dependencies
uv sync --upgrade

# Update specific package
uv add --upgrade numpy
```

### Environment Management

```bash
# Create virtual environment (automatic with uv sync)
uv venv

# Activate environment (usually not needed with uv run)
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows

# Show environment info
uv pip list
uv pip show package-name
```

## Makefile Integration

The project includes a Makefile where every target goes through `uv run`, so it uses the project's locked environment.

📚 **See the complete Makefile template: [python/templates/Makefile](../templates/Makefile)**

### Make Targets

```bash
# Setup
make install        # uv sync --dev, then install commit and pre-push git hooks
make upgrade        # Upgrade the lockfile to the latest allowed versions
make lock-check     # Fail if uv.lock is out of date

# Quality
make format         # ruff format + ruff check --fix
make lint           # ruff check + ruff format --check (no changes)
make typecheck      # mypy on src/

# Testing
make test           # Run tests
make coverage       # Tests with coverage (floor is fail_under in pyproject.toml)

# Security
make audit          # pip-audit on the locked dependencies
make secrets        # gitleaks (needs gitleaks installed)

# Everything CI runs
make check          # lock-check, lint, typecheck, coverage

# Run and build
make run            # python -m <package>
make build          # uv build
make clean          # Remove build output and caches
```

Run `make help` for the list.

## CI/CD Integration

See [GitHub Actions Setup](./cicd/GITHUB_ACTIONS.md) for the workflow templates. The uv steps in them look like this:

```yaml
- name: Install uv
  uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
  with:
    enable-cache: true

- name: Install dependencies
  run: uv sync --dev

- name: Run tests
  run: uv run pytest --cov
```

CI sets `UV_FROZEN: "1"`, so a stale or missing `uv.lock` fails the build rather than being silently re-resolved.

## Migration from pip

### For Developers

Replace pip commands with uv equivalents:

| pip command | uv command |
|------------|------------|
| `pip install package` | `uv add package` |
| `pip install -e .` | `uv sync` |
| `pip install -r requirements.txt` | `uv pip install -r requirements.txt` |
| `pip uninstall package` | `uv remove package` |
| `pip list` | `uv pip list` |
| `pip freeze` | `uv pip freeze` |
| `python script.py` | `uv run python script.py` |
| `pytest` | `uv run pytest` |

### For Scripts

Update automation scripts:

```bash
# Old (pip)
pip install -e ".[dev]"
pytest

# New (uv)
uv sync --dev
uv run pytest
```

## Troubleshooting

### Common Issues

**uv command not found**
```bash
# Reinstall uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Add to PATH
export PATH="$HOME/.cargo/bin:$PATH"
```

**Dependency conflicts**
```bash
# Clear cache and reinstall
uv cache clean
rm -rf .venv uv.lock
uv sync --dev
```

**Wrong Python version**
```bash
# Specify Python version (match requires-python in pyproject.toml)
uv venv --python 3.13
uv sync
```

**CI fails with a lockfile error**
`uv.lock` is out of date or not committed. Run `uv lock`, commit the result, and check with `make lock-check`. CI sets `UV_FROZEN=1` on purpose.

**Import errors after install**
```bash
# Ensure using uv run
uv run python  # Not just 'python'
uv run pytest  # Not just 'pytest'
```

## Best Practices

### DO's
✅ Always use `uv run` to execute commands  
✅ Commit `uv.lock` to version control  
✅ Use `uv add` instead of editing pyproject.toml manually  
✅ Keep dev dependencies separate with `--dev` flag  
✅ Use `uv sync` after pulling changes  

### DON'Ts
❌ Don't use pip alongside uv in the same project  
❌ Don't manually activate venv when using `uv run`  
❌ Don't ignore uv.lock in git  
❌ Don't edit uv.lock manually  
❌ Don't mix pip and uv commands  

## Advanced Usage

### Custom Package Index
```bash
# Add custom index
uv add --index-url https://custom.pypi.org/simple/ package

# Use extra index
uv add --extra-index-url https://extra.pypi.org/simple/ package
```

### Platform-specific Dependencies
Use PEP 508 environment markers in `[project].dependencies`:
```toml
[project]
dependencies = [
    "windows-only-package ; sys_platform == 'win32'",
    "unix-only-package ; sys_platform != 'win32'",
]
```

### Workspace Management
```bash
# For monorepo with multiple packages: sync every workspace member
uv sync --all-packages
```

## Related Documentation

- [uv Documentation](https://docs.astral.sh/uv/)
- [pyproject.toml Specification](https://packaging.python.org/en/latest/specifications/pyproject-toml/)
- [Python Packaging Guide](https://packaging.python.org/)
- [Project README](../../README.md)

## Template Metadata

- **Repository**: [vibe-coding-templates](https://github.com/chrishayuk/vibe-coding-templates)
- **Path**: python/docs/PACKAGE_MANAGEMENT.md
- **Version**: 1.0.0
- **Date**: 2025-01-19
- **Author**: chrishayuk