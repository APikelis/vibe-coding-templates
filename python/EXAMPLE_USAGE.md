# Example Usage by AI Agents

This document demonstrates how an AI agent would use these templates to bootstrap a project.

## Example Scenario: User Requests a Data Processing Tool

**User**: "Create a Python project called data-processor that can read CSV files and generate reports"

**AI Agent Response & Actions**:

### Step 1: Acknowledge and Plan

"I'll create a Python project called 'data-processor' with CSV processing capabilities. Let me bootstrap this using the vibe-coding-templates structure with:
- Package management using uv
- Testing with pytest
- CI/CD with GitHub Actions
- Pre-commit hooks for code quality"

### Step 2: Create Project Structure

```bash
# Create directories
mkdir -p data-processor
cd data-processor
mkdir -p src/data_processor tests docs .github/workflows scripts
```

### Step 3: Copy the Configuration Templates

Copy, do not retype. The templates carry the SHA-pinned hooks and strict tool settings.

```bash
# Paths are relative to the vibe-coding-templates checkout
cp <templates>/python/templates/config/pyproject.toml pyproject.toml
cp <templates>/python/templates/config/.gitignore .gitignore
cp <templates>/python/templates/config/.pre-commit-config.yaml .pre-commit-config.yaml
cp <templates>/python/templates/Makefile Makefile
cp <templates>/python/templates/cicd/workflows/ci.yml .github/workflows/ci.yml

# Fill in the placeholders ({{package_name}}, {{description}}, {{python_version}})
sed -i '' 's/{{package_name}}/data_processor/g' pyproject.toml Makefile .github/workflows/ci.yml   # macOS
sed -i '' 's/{{description}}/A tool for reading CSV files and generating reports/' pyproject.toml
sed -i '' 's/{{python_version}}/3.12/' pyproject.toml
```

Note that `pyproject.toml` uses the package name with an underscore for `[tool.hatch]` and coverage paths, and `name = "data_processor"` is normalised by Python tooling to `data-processor`.

Uncomment the script entry in `[project.scripts]`:

```toml
[project.scripts]
data-processor = "data_processor.cli:main"
```

Then add the project's own dependencies. Let uv write the versions:

```bash
uv add pandas click rich pydantic
uv add --dev pandas-stubs     # strict mypy needs stubs for pandas
```

### Step 4: Create Source Files

**src/data_processor/__init__.py**:
```python
"""Data processor package for CSV analysis and reporting."""

__version__ = "0.1.0"
```

`__version__` is required: the CI wheel smoke test imports it.

**src/data_processor/processor.py**:
```python
"""Core data processing functionality."""

from pathlib import Path
from typing import Any

import pandas as pd


class DataProcessor:
    """Process CSV files and generate reports."""

    def __init__(self, file_path: Path) -> None:
        """Initialize with CSV file path."""
        self.file_path = file_path
        self.data: pd.DataFrame | None = None

    def load_data(self) -> pd.DataFrame:
        """Load CSV data into DataFrame."""
        self.data = pd.read_csv(self.file_path)
        return self.data

    def generate_summary(self) -> dict[str, Any]:
        """Generate summary statistics."""
        if self.data is None:
            raise ValueError("No data loaded. Call load_data() first.")

        return {
            "rows": len(self.data),
            "columns": len(self.data.columns),
            "column_names": list(self.data.columns),
            "missing_values": self.data.isnull().sum().to_dict(),
            "numeric_summary": self.data.describe().to_dict(),
        }
```

**src/data_processor/cli.py**:
```python
"""Command-line interface for data processor."""

from pathlib import Path

import click
from rich.console import Console
from rich.table import Table

from .processor import DataProcessor

console = Console()


@click.command()
@click.argument("csv_file", type=click.Path(exists=True))
@click.option("--output", "-o", help="Output file for report")
def main(csv_file: str, output: str | None) -> None:
    """Process CSV file and generate report."""
    try:
        processor = DataProcessor(Path(csv_file))
        processor.load_data()
        summary = processor.generate_summary()

        table = Table(title="Data Summary")
        table.add_column("Metric", style="cyan")
        table.add_column("Value", style="green")
        table.add_row("Total Rows", str(summary["rows"]))
        table.add_row("Total Columns", str(summary["columns"]))
        console.print(table)

        if output:
            report = (
                "Data Summary Report\n"
                "==================\n"
                f"Rows: {summary['rows']}\n"
                f"Columns: {summary['columns']}\n"
            )
            Path(output).write_text(report)
            console.print(f"[green]Report saved to {output}[/green]")

    except Exception as e:
        console.print(f"[red]Error: {e}[/red]")
        raise click.ClickException(str(e)) from e


if __name__ == "__main__":
    main()
```

### Step 5: Create Tests

**tests/test_processor.py**:
```python
"""Tests for data processor."""

from pathlib import Path

import pandas as pd
import pytest

from data_processor.processor import DataProcessor


@pytest.fixture
def sample_csv(tmp_path: Path) -> Path:
    """Create a sample CSV file for testing."""
    csv_path = tmp_path / "test_data.csv"
    df = pd.DataFrame(
        {
            "name": ["Alice", "Bob", "Charlie"],
            "age": [25, 30, 35],
            "city": ["NYC", "LA", "Chicago"],
        }
    )
    df.to_csv(csv_path, index=False)
    return csv_path


def test_load_data(sample_csv: Path) -> None:
    """Test loading CSV data."""
    processor = DataProcessor(sample_csv)
    data = processor.load_data()

    assert data is not None
    assert len(data) == 3
    assert list(data.columns) == ["name", "age", "city"]


def test_generate_summary(sample_csv: Path) -> None:
    """Test generating summary statistics."""
    processor = DataProcessor(sample_csv)
    processor.load_data()
    summary = processor.generate_summary()

    assert summary["rows"] == 3
    assert summary["columns"] == 3
    assert "name" in summary["column_names"]


def test_summary_without_loading_raises() -> None:
    """Test that summary raises error if data not loaded."""
    processor = DataProcessor(Path("dummy.csv"))

    with pytest.raises(ValueError, match="No data loaded"):
        processor.generate_summary()
```

Tests import `data_processor`, not `src.data_processor`: `uv sync` installs the package, so the src layout resolves normally.

### Step 6: GitHub Actions Workflow

Already copied in Step 3 as `.github/workflows/ci.yml`. It runs lint, type check, the test matrix with the coverage floor from `pyproject.toml`, a gitleaks scan, a dependency audit and a wheel smoke test. For a CLI tool that is never published, that is all it needs: do not copy `release.yml`, `docs.yml` or `benchmark.yml`.

Adjust the Python matrix in `ci.yml` to the versions this project supports.

### Step 7: Pre-commit Configuration

Already copied in Step 3. If a dependency needs type stubs in the mypy hook, add them to the dev group (`uv add --dev <package>-stubs`) rather than to the hook, because the hook runs mypy through `uv run`.

### Step 8: Create README.md

````markdown
# data-processor

A tool for reading CSV files and generating reports.

## Features

- Load and analyze CSV files
- Generate summary statistics
- Command-line interface with rich output

## Installation

```bash
# Install uv package manager
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies and git hooks
make install
```

## Usage

```bash
# Process a CSV file
uv run data-processor data.csv

# Save report to file
uv run data-processor data.csv --output report.txt
```

## Development

```bash
make format       # Format and apply safe lint fixes
make check        # Everything CI runs: lockfile, lint, types, tests with coverage
```

## Project Structure

```text
data-processor/
├── src/data_processor/
│   ├── __init__.py
│   ├── processor.py
│   └── cli.py
├── tests/
│   └── test_processor.py
├── pyproject.toml
├── uv.lock
├── Makefile
├── README.md
└── .github/workflows/
    └── ci.yml
```
````

### Step 9: Initialize and Test

```bash
# Install dependencies (creates uv.lock, which is committed)
uv sync --dev

# Initialize git FIRST (required for pre-commit)
git init

# Install pre-commit hooks (commit and pre-push)
uv run pre-commit install --install-hooks
uv run pre-commit install --hook-type pre-push

# Fix any initial formatting and lint issues
make format

# Everything CI runs
make check

# Stage, let the hooks run, then commit
git add .
uv run pre-commit run --all-files   # may need a second run if files were fixed
git add .
git commit -m "Initial commit: Data processor project structure"
```

### Step 10: Verification Output

```text
✅ Project structure created
✅ Dependencies installed with uv (uv.lock committed)
✅ Tests passing (3 tests), coverage at or above the 80% floor
✅ ruff and mypy (strict) clean
✅ Pre-commit hooks installed (commit and pre-push)
✅ GitHub Actions CI configured
✅ Git repository initialized

Your data-processor project is ready! You can now:
1. Add more CSV processing features to src/data_processor/
2. Write additional tests in tests/
3. Push to GitHub to trigger CI
4. Run 'uv run data-processor --help' to see CLI options
```

## Key Takeaways for AI Agents

1. **Always follow BOOTSTRAP.md** - It provides the exact steps
2. **Copy templates, do not retype them** - They carry SHA pins and strict settings that a hand-written file loses
3. **Customize for user needs** - Added pandas, click, rich for CSV processing
4. **Include domain-specific code** - Created processor.py with actual functionality
5. **Write meaningful, typed tests** - Tests cover the actual features
6. **Verify everything works** - Run `make check` before declaring success

## Template Adaptation Points

When using these templates, AI agents should:

- **Identify project type** (CLI tool, web service, library, etc.)
- **Select appropriate dependencies** with `uv add` (pandas for data, fastapi for web, etc.), and the matching `*-stubs` packages in the dev group
- **Create relevant source files** (not just hello_world)
- **Write domain-specific tests** (test actual functionality)
- **Trim the workflows** - keep `ci.yml`; add `release.yml`, `docs.yml` or `benchmark.yml` only when the project needs them
- **Update documentation** (README should describe actual project)
