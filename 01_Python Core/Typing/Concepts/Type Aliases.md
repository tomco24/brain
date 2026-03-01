---
type: concept
tags: [python, typing, type-alias, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Type Aliases

## Overview
Type aliases give a readable name to complex or repetitive type expressions. Python 3.12 introduces the `type` statement as the canonical, explicit way to define aliases — replacing the older `TypeAlias` annotation.

## Key Points
- **`type` statement (3.12+)**: `type Vector = list[float]` — creates a `TypeAliasType` object, evaluated lazily (supports forward references).
- **`TypeAlias` (3.10–3.11)**: `Vector: TypeAlias = list[float]` — explicit marker for older versions.
- **Simple assignment (pre-3.10)**: `Vector = list[float]` — works but mypy may struggle to distinguish aliases from variables.
- Aliases are **transparent** to type checkers — `Vector` and `list[float]` are identical types.
- **Generic aliases** (3.12): `type Matrix[T] = list[list[T]]` — built-in generic support in the statement.
- Aliases can reference each other and be recursive (via the lazy evaluation of `type`).

## When to Use
- Long, repeated type expressions: `type JsonValue = str | int | float | bool | None | dict[str, "JsonValue"] | list["JsonValue"]`
- Domain clarity: `type UserId = int` (documents intent; NOT a newtype — still structurally `int`)
- Callback/callable signatures: `type Handler = Callable[[Request], Response]`
- Simplifying deeply generic types in function signatures

## When NOT to Use
- When you want nominal typing (use `NewType` instead — creates a distinct type at type-check time)
- Simple one-off generics where the full expansion is just as readable
- Replacing proper `Protocol` / `TypedDict` definitions — aliases name types, not define structures

## Examples

```python
# ✅ Python 3.12+ — preferred
type Vector = list[float]
type Matrix[T] = list[list[T]]
type JsonValue = str | int | float | bool | None | dict[str, "JsonValue"] | list["JsonValue"]
type Handler = Callable[[Request], Awaitable[Response]]

# ✅ Python 3.10+ explicit marker
from typing import TypeAlias
Vector: TypeAlias = list[float]

# ✅ NewType for nominal distinction (not an alias)
from typing import NewType
UserId = NewType("UserId", int)
user_id = UserId(42)
plain_int: int = user_id  # OK — NewType is a subtype
# user_id = 42  # ❌ mypy error: int is not UserId

# ✅ Generic alias (3.12)
type Pair[T, U] = tuple[T, U]
coords: Pair[float, float] = (1.0, 2.0)
```

## Related Concepts
- [[TypeVar and Generics]]
- [[Protocol]]
- [[TypedDict]]

## References
- [PEP 695 – Type Parameter Syntax](https://peps.python.org/pep-0695/)
- [typing — `TypeAlias`](https://docs.python.org/3/library/typing.html#typing.TypeAlias)
- [mypy: Type aliases](https://mypy.readthedocs.io/en/stable/type_aliases.html)
