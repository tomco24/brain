---
type: concept
tags: [python, typing, generics, typevar, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# TypeVar and Generics

## Overview
Generics allow you to write functions and classes that work correctly with multiple types while still being fully type-safe. Python 3.12 introduces native syntax (`[T]`) that eliminates most boilerplate.

## Key Points

### Python 3.12+ Syntax (Preferred)
- `def first[T](seq: Sequence[T]) -> T:` — no `TypeVar` declaration needed
- `class Stack[T]:` — generic class without `Generic[T]` base
- `type Pair[T, U] = tuple[T, U]` — generic alias

### Classic Syntax (3.9–3.11)
- `T = TypeVar("T")` — must be defined before use
- `class Stack(Generic[T]):` — explicit `Generic` base
- `TypeVar` options: `bound=`, `constraints=`, `covariant=`, `contravariant=`

### Bounds and Constraints
- **Bound**: `T = TypeVar("T", bound=Comparable)` — T must be a subtype of `Comparable`
- **Constraints**: `T = TypeVar("T", int, str)` — T can only be exactly `int` or `str`
- Prefer bounds (more flexible); use constraints only when you need a closed set

### Variance (3.12+ new style)
- `[T_co: covariant]` — covariant
- `[T_contra: contravariant]` — contravariant
- Default is invariant

## When to Use
- Functions that return the same type they receive (`identity`, `map`, `filter`, `first`)
- Container classes (`Stack[T]`, `Queue[T]`, `Result[T, E]`)
- Mixins or utilities that operate on a family of types
- Repository or service layers that are type-agnostic

## When NOT to Use
- When you need structural subtyping without inheritance → use [[Protocol]]
- When the type is always the same (no parameterization needed)
- Overuse creates complexity — prefer concrete types when the generic adds no value

## Examples

```python
# ✅ Python 3.12+ — clean generic syntax
from collections.abc import Sequence

def first[T](seq: Sequence[T]) -> T:
    return seq[0]

class Stack[T]:
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

# ✅ Bounded TypeVar (3.12+)
from typing import Protocol

class Comparable(Protocol):
    def __lt__(self, other: "Comparable") -> bool: ...

def minimum[T: Comparable](seq: Sequence[T]) -> T:
    return min(seq)  # type: ignore[arg-type]  # min needs SupportsLessThan

# ✅ Classic syntax (still valid in 3.12)
from typing import TypeVar, Generic
T = TypeVar("T")

class Box(Generic[T]):
    def __init__(self, value: T) -> None:
        self.value = value

    def unwrap(self) -> T:
        return self.value

# ✅ Result type (error handling without exceptions)
type Result[T, E] = tuple[T, None] | tuple[None, E]

def divide(a: float, b: float) -> Result[float, str]:
    if b == 0:
        return None, "division by zero"
    return a / b, None
```

## Related Concepts
- [[Type Aliases]]
- [[Protocol]]
- [[Variance Patterns]]
- [[ParamSpec and Concatenate]]
- [[Unpack and TypeVarTuple]]

## References
- [PEP 695 – Type Parameter Syntax (3.12)](https://peps.python.org/pep-0695/)
- [mypy: Generics](https://mypy.readthedocs.io/en/stable/generics.html)
- [Python docs: `TypeVar`](https://docs.python.org/3/library/typing.html#typing.TypeVar)
