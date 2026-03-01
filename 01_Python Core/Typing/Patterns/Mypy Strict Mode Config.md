---
type: pattern
category: code-pattern
tags: [python, typing, mypy, strict, configuration, pyproject]
created: 2026-03-01
modified: 2026-03-01
complexity: beginner
---

# Mypy Strict Mode Config

## Problem
mypy has permissive defaults that allow many type errors to pass silently. Without strict configuration, the type checker provides little value — catching only the most obvious mistakes.

## Solution
Enable `--strict` mode (or equivalent per-flag settings in config) and understand what each flag does. Configure per-module overrides for third-party libraries without stubs.

## Structure

```toml
# pyproject.toml — recommended
[tool.mypy]
python_version = "3.12"
strict = true
```

## Full Example

### `pyproject.toml` (recommended)

```toml
[tool.mypy]
python_version = "3.12"

# ── Strict mode (enables all checks below) ──────────────────────────────────
strict = true

# ── What 'strict = true' enables (explicit for documentation) ───────────────
warn_unused_configs = true
disallow_any_generics = true       # No bare list, dict, etc. — must be list[int]
disallow_subclassing_any = true    # No class Foo(Any)
disallow_untyped_calls = true       # Can't call untyped functions from typed code
disallow_untyped_defs = true        # All functions must be annotated
disallow_incomplete_defs = true     # Partial annotations not allowed
check_untyped_defs = true           # Also type-check unannotated functions
disallow_untyped_decorators = true  # Decorators must be typed
warn_return_any = true              # Warn when returning Any
warn_unused_ignores = true          # Warn on unnecessary # type: ignore
no_implicit_reexport = true         # Must explicitly re-export with __all__
strict_equality = true              # Warn on impossible == comparisons

# ── Additional recommended settings ─────────────────────────────────────────
warn_redundant_casts = true         # Warn on unnecessary cast()
extra_checks = true                 # Enables --warn-incomplete-stub etc.

# ── Per-module overrides for untyped third-party libs ───────────────────────
[[tool.mypy.overrides]]
module = [
    "boto3.*",
    "botocore.*",
    "celery.*",
]
ignore_missing_imports = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false       # Allow loose typing in tests
```

### `mypy.ini` (alternative)

```ini
[mypy]
python_version = 3.12
strict = True
warn_redundant_casts = True
warn_unused_ignores = True
extra_checks = True

[mypy-boto3.*]
ignore_missing_imports = True

[mypy-tests.*]
disallow_untyped_defs = False
```

### Running mypy

```bash
# Check entire project
mypy src/

# Check a single file
mypy src/module.py

# With explicit config
mypy --config-file pyproject.toml src/

# Show error codes (useful for targeted # type: ignore[code])
mypy --show-error-codes src/

# Generate a baseline (ignore existing errors, catch new ones)
mypy --baseline-file .mypy-baseline src/
```

### Common `# type: ignore` patterns

```python
# Narrow suppression — always specify the error code
result = some_untyped_lib()  # type: ignore[no-any-return]

# Importing untyped third-party module
import untyped_module  # type: ignore[import-untyped]

# Override return for a known-safe dynamic pattern
def factory(cls: type[T]) -> T:
    return cls()  # type: ignore[call-arg]  # cls always has no-arg __init__
```

### Useful stub packages

```bash
pip install types-requests types-PyYAML types-redis types-boto3
# Or via pip install boto3-stubs[s3,dynamodb]
```

## Links and resources
- [[TypeVar and Generics]]
- [[Protocol]]
- [[Annotated]]
- [mypy strict docs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-strict)
- [mypy error codes](https://mypy.readthedocs.io/en/stable/error_code_list.html)
- [typeshed (stub packages)](https://github.com/python/typeshed)
