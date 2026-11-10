# Makefile for Python projects using uv.
# Replace {{package_name}}. Run `make help` for targets.
# Every target goes through `uv run`, so it uses the project's locked environment.

PACKAGE_NAME := {{package_name}}
SRC_DIR := src
TEST_DIR := tests

.DEFAULT_GOAL := help

.PHONY: help
help:  ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  %-18s %s\n", $$1, $$2}'

# --- Setup -------------------------------------------------------------------
.PHONY: install
install:  ## Install dependencies and git hooks
	uv sync --dev
	uv run pre-commit install --install-hooks
	uv run pre-commit install --hook-type pre-push

.PHONY: upgrade
upgrade:  ## Upgrade the lockfile to the latest allowed versions
	uv lock --upgrade
	uv sync --dev

.PHONY: lock-check
lock-check:  ## Fail if uv.lock is out of date
	uv lock --check

# --- Quality -----------------------------------------------------------------
.PHONY: format
format:  ## Format code and apply safe lint fixes
	uv run ruff format .
	uv run ruff check . --fix

.PHONY: lint
lint:  ## Lint and check formatting (no changes)
	uv run ruff check .
	uv run ruff format --check .

.PHONY: typecheck
typecheck:  ## Type check with mypy
	uv run mypy $(SRC_DIR)

.PHONY: test
test:  ## Run tests
	uv run pytest

.PHONY: coverage
coverage:  ## Run tests with coverage (floor is set in pyproject.toml)
	uv run pytest --cov --cov-report=term-missing --cov-report=html

.PHONY: audit
audit:  ## Scan locked dependencies for known vulnerabilities
	uv export --frozen --no-emit-project --format requirements-txt -o .requirements-audit.txt
	uvx pip-audit@2.10.1 --requirement .requirements-audit.txt --disable-pip
	rm -f .requirements-audit.txt

.PHONY: secrets
secrets:  ## Scan the working tree and history for secrets (needs gitleaks installed)
	gitleaks git --redact

.PHONY: check
check: lock-check lint typecheck coverage  ## Everything CI runs, locally

# --- Run and build -----------------------------------------------------------
.PHONY: run
run:  ## Run the package
	uv run python -m $(PACKAGE_NAME)

.PHONY: build
build:  ## Build sdist and wheel
	uv build

# --- Housekeeping ------------------------------------------------------------
.PHONY: clean
clean:  ## Remove build output and caches
	rm -rf build dist *.egg-info .pytest_cache .mypy_cache .ruff_cache htmlcov .coverage coverage.xml
	find . -type d -name __pycache__ -not -path "./.venv/*" -exec rm -rf {} +
