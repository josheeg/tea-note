# tea-note
setup test tea by tea.md
#!/usr/bin/env python3
"""TEA (Testing, Enforcement, and Architecture) Standard Project Bootstrapper.

Bootstraps a repository configured with `uv`, Ruff, MyPy, Pytest with coverage,
and a strict TEA layout (Schemas -> Tests -> Source).
"""

from pathlib import Path
import subprocess
import sys


def create_file(path: Path, content: str) -> None:
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text(content.strip() + "\n", encoding="utf-8")
    print(f"  [created] {path}")


def main() -> None:
    root = Path.cwd()
    print(f"🚀 Initializing TEA Project in: {root}\n")

    # 1. Directory Structure
    dirs = [
        root / "src" / "schemas",
        root / "tests",
        root / ".github" / "workflows",
    ]
    for d in dirs:
        d.mkdir(parents=True, exist_ok=True)

    # 2. pyproject.toml configuration
    pyproject_content = """
[project]
name = "tea-project"
version = "0.1.0"
description = "A TEA Standard compliant project"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
    "pydantic>=2.0.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "pytest-cov>=5.0.0",
    "mypy>=1.10.0",
    "ruff>=0.4.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "N"]
ignore = []

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
minversion = "8.0"
addopts = "-ra -q --cov=src --cov-report=term-missing"
testpaths = ["tests"]
pythonpath = ["."]
"""
    create_file(root / "pyproject.toml", pyproject_content)

    # 3. Source & Schema Boilerplate
    create_file(root / "src" / "__init__.py", "")
    create_file(root / "src" / "schemas" / "__init__.py", "")

    sample_schema = """
from pydantic import BaseModel, ConfigDict, Field


class BaseContract(BaseModel):
    model_config = ConfigDict(strict=True, frozen=True)


class ExamplePayload(BaseContract):
    id: int = Field(..., description="Unique identifier")
    name: str = Field(..., description="Name associated with payload")
"""
    create_file(root / "src" / "schemas" / "example.py", sample_schema)

    sample_core = """
from src.schemas.example import ExamplePayload


def process_payload(payload: ExamplePayload) -> str:
    return f"Processed #{payload.id}: {payload.name}"
"""
    create_file(root / "src" / "core.py", sample_core)

    # 4. Tests Boilerplate
    create_file(root / "tests" / "__init__.py", "")

    sample_test = """
import pytest
from pydantic import ValidationError
from src.schemas.example import ExamplePayload
from src.core import process_payload


def test_example_payload_schema_valid() -> None:
    payload = ExamplePayload(id=1, name="TEA Unit")
    assert payload.id == 1
    assert payload.name == "TEA Unit"


def test_example_payload_schema_invalid() -> None:
    with pytest.raises(ValidationError):
        ExamplePayload(id="invalid_id", name="TEA Unit")  # type: ignore[arg-type]


def test_process_payload() -> None:
    payload = ExamplePayload(id=42, name="Autonomous Agent")
    result = process_payload(payload)
    assert result == "Processed #42: Autonomous Agent"
"""
    create_file(root / "tests" / "test_example.py", sample_test)

    # 5. README & Quality Gate Script
    readme_content = """
# TEA Standard Project

This project adheres to the **TEA (Testing, Enforcement, Architecture)** Standard.

## Quality Gate Executable Sequence

Run the complete validation loop before completing any task:

```bash
uv run ruff check .
uv run mypy src/ tests/
uv run pytest --cov=src
