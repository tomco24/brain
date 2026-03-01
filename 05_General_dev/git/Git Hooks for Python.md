# Git Hooks for Python

## What Are Git Hooks

Scripts that run automatically at certain points in git workflow. Located in `.git/hooks/`.

## Pre-commit Hook

### Installation

```bash
# Install pre-commit
pip install pre-commit

# Create .pre-commit-config.yaml
```

### Basic Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/psf/black
    rev: 24.1.0
    hooks:
      - id: black
        language_version: python3.9

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

### Usage

```bash
# Install hooks
pre-commit install

# Run on all files
pre-commit run --all-files

# Run on staged files
pre-commit run

# Update hook versions
pre-commit autoupdate
```

## Python-Specific Hooks

### Ruff (Linter + Formatter)

```yaml
- repo: https://github.com/astral-sh/ruff-pre-commit
  rev: v0.1.0
  hooks:
    - id: ruff
      args: [--fix, --exit-non-zero-on-fix]
    - id: ruff-format
```

### Black (Formatter)

```yaml
- repo: https://github.com/psf/black
  rev: 24.1.0
  hooks:
    - id: black
      language_version: python3.11
```

### MyPy (Type Checker)

```yaml
- repo: https://github.com/pre-commit/mirrors-mypy
  rev: v1.8.0
  hooks:
    - id: mypy
      additional_dependencies: [types-all]
      args: [--ignore-missing-imports]
```

### Pytest

```yaml
- repo: https://github.com/pytest-dev/pytest-pre-commit
  rev: v0.0.1
  hooks:
    - id: pytest
      args: [--doctest-modules, --ignore=tests/]
```

## Git Hooks for Testing

### Pre-commit Testing

```yaml
repos:
  - repo: local
    hooks:
      - id: pytest-checks
        name: Run pytest
        entry: pytest
        args: [--doctest-modules]
        language: system
        pass_filenames: false
        types: [python]
```

### Pre-push Hook

```yaml
# .pre-commit--config.yaml
- repo: local
  hooks:
    - id: run-tests
      name: Run tests
      entry: pytest
      args: [tests/]
      language: system
      stages: [push]
```

## Custom Hooks

### Custom Python Script Hook

```bash
#!/usr/bin/env python3
# .git/hooks/pre-commit
import subprocess
import sys

# Run tests
result = subprocess.run(['pytest', '--tb=short'], capture_output=True)

if result.returncode != 0:
    print("Tests failed!")
    sys.exit(1)
```

### Make Executable

```bash
chmod +x .git/hooks/pre-commit
```

## GitHub Actions Integration

### Basic CI

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -e .[dev]
          pip install pre-commit
      
      - name: Run pre-commit
        run: pre-commit run --all-files
      
      - name: Run tests
        run: pytest
```

### Matrix Testing

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11', '3.12']
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Run tests
        run: |
          pip install pytest
          pytest
```

## Local Hook Installation

```bash
# Install to project
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit

# Or use pre-commit framework
pre-commit install
pre-commit install -t pre-push
pre-commit install -t commit-msg
```

## Hook Configuration Options

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
        stages: [commit, push]  # when to run
        types: [python]  # file types
        exclude: '^tests/'  # exclude pattern
```

## Commit Message Hook

### Commitlint

```yaml
- repo: https://github.com/conventional-changelog/commitlint
  rev: v18.6.0
  hooks:
    - id: commitlint
      stages: [commit-msg]
      additional_args: '--config .commitlintrc.json'
```

### Commitizen

```bash
# Install
pip install commitizen

# Use
cz commit
cz changelog
```
