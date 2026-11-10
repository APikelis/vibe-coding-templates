# Python Project Bootstrap Guide for AI Agents

This guide helps AI agents bootstrap new Python projects using the vibe-coding-templates.

## ⚠️ REQUIRED Components

**Every Python project MUST include:**
1. ✅ Project structure with src/ and tests/
2. ✅ Package management with uv
3. ✅ Testing with pytest
4. ✅ **GitHub Actions workflows** (.github/workflows/ci.yml)
5. ✅ **Pre-commit hooks** (.pre-commit-config.yaml)
6. ✅ Git repository initialization

## Quick Start Checklist

```markdown
☐ 1. Create project structure (src/, tests/, docs/, .github/workflows/)
☐ 2. Copy pyproject.toml from templates/config/ and fill in the placeholders
☐ 3. Create source and test files
☐ 4. Create Makefile and .gitignore
☐ 5. CREATE GitHub Actions workflow (.github/workflows/ci.yml) - REQUIRED
☐ 6. CREATE pre-commit configuration (.pre-commit-config.yaml) - REQUIRED
☐ 7. Initialize git and install dependencies
☐ 8. Run ALL verification tests
```

## Step 1: Gather Project Information

Required information:
- **Project name**: Directory name (e.g., "my-awesome-project")
- **Package name**: Python package name (e.g., "my_awesome_project")
- **Description**: Brief project description
- **Python version**: Minimum Python version (default: 3.12, matching the CI matrix in `ci.yml`)

## Step 2: Create Project Structure

```bash
mkdir -p {{project_name}}
cd {{project_name}}
mkdir -p src/{{package_name}} tests docs scripts .github/workflows
```

## Step 3: Create Core Configuration Files

### pyproject.toml

⚠️ **IMPORTANT: Read `docs/PACKAGE_MANAGEMENT.md` AND `docs/testing/CODE_QUALITY.md` BEFORE implementing!**
These documents contain critical details about:
- Package management with uv
- Code quality tools configuration (Ruff, mypy)
- Best practices for tool integration

Copy the template. It already holds the build system, dev dependency group, pytest, coverage (80% floor), strict mypy and the ruff rule set:

```bash
cp ../templates/config/pyproject.toml pyproject.toml
```

Replace `{{package_name}}`, `{{description}}` and `{{python_version}}` (see the variables table below).

**Dev tools**: the template declares them in `[dependency-groups]` (PEP 735) without version pins. Do not hand-pin versions, and do not use the deprecated `[tool.uv] dev-dependencies`. Run:

```bash
uv add --dev pytest pytest-cov mypy ruff pre-commit
```

uv writes the current versions and creates `uv.lock`.

**REQUIRED ACTIONS**:
1. Read `docs/PACKAGE_MANAGEMENT.md` for package management details
2. Read `docs/testing/CODE_QUALITY.md` for tool configuration (Ruff, mypy)
3. Do not add black or isort; ruff replaces them

### Makefile

📚 **Use the comprehensive Makefile template: [templates/Makefile](templates/Makefile)**

Copy the Makefile template to your project and replace `{{package_name}}` with your actual package name.

```bash
# Copy the Makefile template
cp ../templates/Makefile .

# Replace placeholder with your package name (example for macOS/Linux)
sed -i '' 's/{{package_name}}/your_package_name/g' Makefile  # macOS
# OR
sed -i 's/{{package_name}}/your_package_name/g' Makefile     # Linux
```

The Makefile includes all common development tasks:
- **Setup**: `make install` (dependencies and git hooks), `make upgrade`, `make lock-check`
- **Quality**: `make format`, `make lint`, `make typecheck`
- **Testing**: `make test`, `make coverage`
- **Security**: `make audit` (locked dependencies), `make secrets` (gitleaks)
- **Everything CI runs**: `make check`
- **Run and build**: `make run`, `make build`, `make clean`
- Run `make help` to see all targets

📚 **See also**: [docs/PACKAGE_MANAGEMENT.md#makefile-integration](docs/PACKAGE_MANAGEMENT.md) for detailed documentation.

### .gitignore

```bash
cp ../templates/config/.gitignore .gitignore
```

Standard Python gitignore. `uv.lock` is **committed on purpose** for reproducible installs; CI runs with `UV_FROZEN=1` and needs it.

## Step 4: Create Documentation and Initial Source Files

### README.md
```markdown
# {{project_name}}

{{description}}

## Installation

```bash
uv sync --dev
```

## Usage

```bash
uv run {{package_name}}
```

## Development

```bash
make test         # Run tests
make lint         # Run linting
make format       # Format code
make check        # Run all checks
```
```

### llms.txt (AI Agent Documentation)

📚 **Use the llms.txt template: [templates/llms.txt](templates/llms.txt)**  
📚 **Documentation: [docs/LLMS_TXT.md](docs/LLMS_TXT.md)**

Create an `llms.txt` file to help AI agents understand your project:

```bash
# Copy the template
cp ../templates/llms.txt .

# Edit and replace placeholders with your project details
# See docs/LLMS_TXT.md for detailed guidance on each section
```

The llms.txt file provides AI agents with:
- Project overview and key features
- Quick start code examples
- Project structure visualization
- Common development commands
- Key API references

**Note**: Read [docs/LLMS_TXT.md](docs/LLMS_TXT.md) for best practices and examples.

### CLAUDE.md (Claude Code Instructions)

📚 **Use the CLAUDE.md template: [templates/CLAUDE.md](templates/CLAUDE.md)**

Create a `CLAUDE.md` file with project-specific instructions for Claude Code:

```bash
# Copy the template
cp ../templates/CLAUDE.md .

# Replace placeholders with your project details
sed -i '' 's/{{project_name}}/your_project_name/g' CLAUDE.md  # macOS
sed -i '' 's/{{package_name}}/your_package_name/g' CLAUDE.md
# Add any project-specific notes
```

The CLAUDE.md file helps Claude Code understand:
- Project standards and guidelines
- Common development commands
- Testing requirements
- Code style preferences
- Git workflow

### src/{{package_name}}/__init__.py
```python
"""{{package_name}} package."""
__version__ = "0.1.0"
```

### src/{{package_name}}/main.py
Create a minimal main module with at least one function to test.

## Step 5: Set Up Testing

**BEFORE PROCEEDING**: Read BOTH documents:
1. `docs/testing/TEST_COVERAGE.md` - Testing setup and coverage targets
2. `docs/testing/CODE_QUALITY.md` - Code quality tools configuration

These documents cover:
- Testing best practices and coverage configuration
- Ruff configuration (linting, formatting, import sorting, security rules)
- mypy setup for strict type checking

### tests/test_main.py
Create basic tests that import and test your package:
```python
from src.{{package_name}}.main import your_function

def test_your_function():
    assert your_function() is not None
```

### tests/conftest.py
**ACTION**: Read `docs/testing/TEST_COVERAGE.md` and `docs/testing/UNIT_TESTING.md` for pytest fixtures and testing patterns.
Add pytest fixtures as needed based on the documentation.

## Step 6: Configure GitHub Actions (REQUIRED)

**⚠️ IMPORTANT: Always create GitHub Actions workflows for CI/CD**

Copy the template; do not retype it. Its actions are pinned to commit SHAs, which a hand-written workflow would lose:

```bash
cp ../templates/cicd/workflows/ci.yml .github/workflows/ci.yml
```

Then:
1. Replace `{{package_name}}` (used in the wheel smoke test)
2. Adjust the Python version matrix to what the project supports
3. Commit `uv.lock`: CI runs with `UV_FROZEN=1` and fails if it is missing or out of date

`ci.yml` runs lint and type check, the test matrix with coverage, a gitleaks scan, a dependency audit and a build.

### Optional workflows

Copy these only when the project needs them (each file says at the top when to delete it):

| Template | Needed when |
|----------|-------------|
| `release.yml` | Publishing to PyPI on a `v*` tag (trusted publishing, no stored token) |
| `docs.yml` | Publishing MkDocs to GitHub Pages |
| `benchmark.yml` | The project has `tests/performance/` |

For a proof of concept that must not publish, copy `ci.yml` only.

Read `docs/cicd/GITHUB_ACTIONS.md` for the job-by-job description, PyPI trusted-publisher setup and troubleshooting.

## Step 7: Configure Pre-commit Hooks (REQUIRED)

**⚠️ IMPORTANT: Always set up pre-commit hooks for code quality**

Copy the template; do not retype it. Its `rev` values are full commit SHAs, which a hand-written config would lose:

```bash
cp ../templates/config/.pre-commit-config.yaml .pre-commit-config.yaml
```

What it runs:
- **On commit**: file hygiene checks, `ruff-check --fix`, `ruff-format`, gitleaks (secrets), mypy via `uv run`
- **On push**: pytest with the coverage floor from `pyproject.toml`

Update pins later with `uv run pre-commit autoupdate --freeze`.

Read `docs/cicd/PRE_COMMIT.md` for the hook table, how to add hooks, and troubleshooting.

## Step 8: Initialize and Install

```bash
# Navigate to project directory
cd {{project_name}}

# Initialize git FIRST (required for pre-commit)
git init

# Check if uv is installed, install if not present
command -v uv >/dev/null 2>&1 || curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync --dev

# Install pre-commit hooks (commit and pre-push)
uv run pre-commit install --install-hooks
uv run pre-commit install --hook-type pre-push

# Fix any initial linting and formatting issues (ruff does formatting, imports and linting)
uv run ruff format .
uv run ruff check . --fix

# Stage and verify with pre-commit
git add .
uv run pre-commit run --all-files  # May need to run twice if files are fixed

# If pre-commit made changes, stage them again
git add .

# Initial commit
git commit -m "Initial project structure"
```

## Step 9: Verification

**IMPORTANT**: These commands are from `docs/PACKAGE_MANAGEMENT.md` - read that document if any command fails!

### Running the Application

**⚠️ CRITICAL: Always use `uv run python` to execute Python code, NOT plain `python`**

```bash
# ✅ CORRECT - Uses project's virtual environment
uv run python src/{{package_name}}/main.py
uv run python -m {{package_name}}.main

# ❌ INCORRECT - May use system Python or wrong environment
python src/{{package_name}}/main.py  # DO NOT USE
python -m {{package_name}}.main      # DO NOT USE
```

### Verification Commands

Run these commands to verify setup:

```bash
# Run the application (ALWAYS use uv run python)
uv run python src/{{package_name}}/main.py

# Package management
uv --version
uv pip list

# Testing with coverage (the floor is fail_under in pyproject.toml)
uv run pytest --cov --cov-report=term-missing

# ⚠️ Coverage Requirements (from docs/testing/TEST_COVERAGE.md):
# - Minimum overall: 80%
# - Critical modules: 90-100%
# - New code: 95%+
# If coverage is below 80%, add more tests before considering bootstrap complete

# Code quality (see docs/testing/CODE_QUALITY.md for details)
uv run ruff check .
uv run ruff format --check .
uv run mypy src/

# Pre-commit
uv run pre-commit run --all-files

# Or everything CI runs in one go
make check
```

## Template Variables Reference

| Placeholder | Description | Example |
|------------|-------------|---------|
| `{{project_name}}` | Project directory | `my-project` |
| `{{package_name}}` | Python package | `my_package` |
| `{{description}}` | Project description | `Data processing tool` |
| `{{python_version}}` | Minimum Python version | `3.12` |

## ⚠️ CRITICAL: Read Referenced Documentation

**AI AGENTS MUST READ these documents when performing the related steps:**

- **Package Management** (Step 3, 8, 9): [docs/PACKAGE_MANAGEMENT.md](docs/PACKAGE_MANAGEMENT.md)
  - WHEN: Creating pyproject.toml, Makefile, running uv commands
  - WHY: Contains complete templates, command reference, troubleshooting
  - ⚠️ CRITICAL: Use `uv add --dev` to get latest package versions
  
- **Code Quality** (Step 3, 5, 8, 9): [docs/testing/CODE_QUALITY.md](docs/testing/CODE_QUALITY.md)
  - WHEN: Setting up Ruff and mypy
  - WHY: Rule selection, strict typing, and what ruff replaces
  - ⚠️ CRITICAL: Do not add black or isort; ruff replaces them
  
- **GitHub Actions** (Step 6): [docs/cicd/GITHUB_ACTIONS.md](docs/cicd/GITHUB_ACTIONS.md)
  - WHEN: Setting up CI/CD workflows
  - WHY: Explains matrix testing, caching, workflow optimization
  
- **Pre-commit Hooks** (Step 7): [docs/cicd/PRE_COMMIT.md](docs/cicd/PRE_COMMIT.md)
  - WHEN: Configuring pre-commit hooks
  - WHY: Details hook stages, custom hooks, troubleshooting
  
- **Testing** (Step 5): 
  - [docs/testing/TEST_COVERAGE.md](docs/testing/TEST_COVERAGE.md) - Coverage setup and targets
  - [docs/testing/UNIT_TESTING.md](docs/testing/UNIT_TESTING.md) - Testing patterns and fixtures
  - WHEN: Setting up tests and coverage
  - WHY: Best practices, fixtures, coverage configuration

- **AI Documentation** (Step 4): [docs/LLMS_TXT.md](docs/LLMS_TXT.md)
  - WHEN: Creating llms.txt and CLAUDE.md files
  - WHY: Best practices for AI agent documentation
  - Templates: [templates/llms.txt](templates/llms.txt), [templates/CLAUDE.md](templates/CLAUDE.md)

**DO NOT skip reading these documents - they contain critical implementation details!**

## ⚠️ CRITICAL: Verification Checklist

**DO NOT consider the project complete until ALL of these pass:**

```bash
# 1. Check project structure
ls -la .github/workflows/  # MUST contain ci.yml
ls -la .pre-commit-config.yaml  # MUST exist
ls -la src/ tests/ docs/  # MUST exist

# 2. Verify dependencies install
uv sync --dev  # MUST complete without errors

# 3. Run the application (ALWAYS use uv run python, NOT plain python)
uv run python src/{{package_name}}/main.py  # MUST run without errors

# 4. Run tests with coverage
uv run pytest --cov --cov-report=term-missing  # MUST pass all tests
# The 80% floor (fail_under) is set in pyproject.toml, so this fails below it
# (see docs/testing/TEST_COVERAGE.md). If coverage < 80%, bootstrap is INCOMPLETE - add more tests!

# 5. Check code quality
uv run ruff check .  # MUST pass
uv run ruff format --check .  # MUST pass
uv run mypy src/  # MUST pass (strict mode is on in the template)

# 6. Verify pre-commit hooks
uv run pre-commit run --all-files  # MUST pass

# 7. Everything CI runs, in one command
make check

# 8. Check git
git status  # MUST show initialized repository
```

## Troubleshooting Guide

### Bootstrap Order Issues

The correct order is critical for successful bootstrap:

1. Create project directory structure
2. Create ALL files (including README.md) BEFORE running uv sync
3. Initialize git repository
4. Install dependencies with uv sync --dev
5. Install pre-commit hooks
6. Run pre-commit to fix formatting
7. Commit changes

### Common Bootstrap Failures and Solutions

#### Missing README.md Error
**Problem**: `uv sync` fails with "README.md not found" when readme is referenced in pyproject.toml

**Solution**: Create README.md before running uv sync:
```bash
cat > README.md << 'EOF'
# Project Name
Project description
EOF
```

#### Pre-commit Hook Installation Failures
**Problem**: Pre-commit hooks fail to install or run

**Solutions**:
1. Ensure git is initialized BEFORE installing pre-commit
2. Run `uv sync --dev` to install all dev dependencies first (the mypy and pytest hooks run through `uv run`)
3. Use `pre-push`, not the deprecated stage name `push`
4. Reset: `uv run pre-commit clean && uv run pre-commit install --install-hooks`

#### Dev Dependencies Not Installing
**Problem**: Tools like ruff, mypy, pytest not available after uv sync

**Solution**: Dev tools belong in `[dependency-groups]` (PEP 735), not the deprecated `[tool.uv] dev-dependencies`. Let uv write them:

```bash
uv add --dev pytest pytest-cov mypy ruff pre-commit
```

⚠️ **IMPORTANT**: Always use `uv add --dev` so you get the latest versions rather than hand-pinned ones

#### Directory Navigation Issues
**Problem**: Commands fail with "no such file or directory"

**Solution**: Always cd into project directory after creation:
```bash
mkdir my-project
cd my-project  # Don't forget this!
```

## Common Issues & Fixes

### README.md not found during bootstrap
```bash
# Create README.md before running uv sync
cat > README.md << 'EOF'
# Project Name
Project description
EOF
```

### Linting and formatting errors on first run
```bash
# Ruff owns formatting, import sorting and linting, so there is no tool order to get wrong
uv run ruff format .
uv run ruff check . --fix
```

### Pre-commit hook failures
```bash
# Let pre-commit fix what it can
git add -A
uv run pre-commit run --all-files
# Then commit the fixes
```

### Code Quality Tool Issues

#### Formatter keeps rewriting files
**Problem**: Files change on every run, or lint and format disagree

**Solution**:
- Black and isort are not part of the toolchain. Ruff replaces both
- Remove any `[tool.black]` / `[tool.isort]` tables, black or isort hooks, and editor extensions
- Keep ruff configuration in `pyproject.toml` only (no separate `ruff.toml`)
- See [docs/testing/CODE_QUALITY.md](docs/testing/CODE_QUALITY.md)

#### Mypy Type Stub Issues
**Problem**: mypy reports missing stubs for a third-party package

**Solution**:
- **NEVER use `types-all`** - it's deprecated and causes installation failures
- Add only the specific stubs you need to the dev group: `uv add --dev types-requests`
- The mypy hook runs through `uv run`, so it sees the project's installed stubs
- See [docs/testing/CODE_QUALITY.md](docs/testing/CODE_QUALITY.md) for full mypy configuration guidance

#### CI fails with a lockfile error
**Problem**: `uv.lock` is out of date, or missing from the repository

**Solution**: run `uv lock`, commit `uv.lock`. CI runs with `UV_FROZEN=1` on purpose.

### Markdown linting errors
- Use language specifiers in code blocks: ` ```python` not ` ``` `
- Or use ` ```text` for non-code content

## Success Criteria

✅ **Project is ONLY complete when:**
- All directories exist (including .github/workflows/)
- GitHub Actions workflow file exists (.github/workflows/ci.yml)
- Pre-commit config exists (.pre-commit-config.yaml)
- All dependencies install successfully
- All tests pass
- All linting passes (after running `ruff --fix`)
- Pre-commit hooks are installed and working
- Git repository is initialized

**Missing any of these means the bootstrap is INCOMPLETE!**