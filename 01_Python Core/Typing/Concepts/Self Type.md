---
type: concept
tags: [python, typing, self, fluent-api, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Self Type

## Overview
`Self` (from `typing`, 3.11+) is a special type that refers to the current class, including subclasses. It solves the long-standing problem of typing methods that return `self` — previously requiring complex `TypeVar` workarounds.

## Key Points
- Available as `from typing import Self` (3.11+) or `from typing_extensions import Self` (backport)
- `Self` in a return annotation means "an instance of whatever concrete class this method is called on"
- Correctly types fluent/builder APIs where subclasses chain methods
- Works in `@classmethod` too — `cls: type[Self]` means the class itself
- **No explicit `TypeVar` needed** — `Self` is handled specially by type checkers
- In mypy strict mode, using `Self` is the correct way to type `__init__`-like patterns and factory methods

## When to Use
- Builder / fluent API methods: `def set_name(self, name: str) -> Self: ...`
- `@classmethod` factory methods: `def from_dict(cls, data: dict[str, str]) -> Self: ...`
- `__copy__`, `__deepcopy__`, `__enter__` returning the instance
- Mixin classes that return `self` to allow chaining in subclasses

## When NOT to Use
- When the method truly returns only the base class type (not subclasses)
- When you specifically want to restrict to the *base* class (use the class name directly)
- In standalone functions (not methods) — `Self` only makes sense in class bodies

## Examples

```python
from typing import Self
from __future__ import annotations  # only needed pre-3.12 for forward refs

# ✅ Fluent builder API
class QueryBuilder:
    def __init__(self) -> None:
        self._filters: list[str] = []
        self._limit: int | None = None

    def where(self, condition: str) -> Self:
        self._filters.append(condition)
        return self

    def limit(self, n: int) -> Self:
        self._limit = n
        return self

    def build(self) -> str:
        where_clause = " AND ".join(self._filters)
        limit_clause = f" LIMIT {self._limit}" if self._limit else ""
        return f"SELECT * WHERE {where_clause}{limit_clause}"

class UserQueryBuilder(QueryBuilder):
    def active_only(self) -> Self:  # Returns UserQueryBuilder, not QueryBuilder
        return self.where("active = true")

q = UserQueryBuilder().active_only().where("age > 18").limit(10).build()
# q is correctly typed as UserQueryBuilder at each step ✅

# ✅ classmethod factory
class Config:
    def __init__(self, data: dict[str, str]) -> None:
        self._data = data

    @classmethod
    def from_env(cls) -> Self:
        import os
        return cls(dict(os.environ))

    @classmethod
    def from_dict(cls, data: dict[str, str]) -> Self:
        return cls(data)

class AppConfig(Config):
    ...

cfg = AppConfig.from_env()  # ✅ inferred as AppConfig, not Config

# ✅ Context manager returning Self
class Connection:
    def __enter__(self) -> Self:
        return self

    def __exit__(self, *args: object) -> None:
        self.close()

    def close(self) -> None: ...

# ✅ Old approach with TypeVar (pre-3.11) — for reference
from typing import TypeVar
T = TypeVar("T", bound="OldBase")

class OldBase:
    def copy(self: T) -> T:  # Cumbersome
        ...
```

## Related Concepts
- [[TypeVar and Generics]]
- [[Protocol]]
- [[Override]]

## References
- [PEP 673 – `Self` type](https://peps.python.org/pep-0673/)
- [Python docs: `Self`](https://docs.python.org/3/library/typing.html#typing.Self)
- [mypy: `Self` type](https://mypy.readthedocs.io/en/stable/self_types.html)
