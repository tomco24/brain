---
type: concept
tags: [python, typing, unpack, typevartuple, variadic, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Unpack and TypeVarTuple

## Overview
`TypeVarTuple` enables **variadic generics** — generics parameterized by an arbitrary number of types. `Unpack` "unpacks" a `TypeVarTuple` into a position (like `*` does at runtime). Together they enable type-safe heterogeneous `*args`, typed `tuple` operations, and tensor-shape typing.

## Key Points
- `Ts = TypeVarTuple("Ts")` — captures a tuple of types: `*Ts` in signatures
- `*args: *Ts` — each element of `args` has its own tracked type
- `Unpack[Ts]` / `*Ts` — both are equivalent in 3.11+; use `*Ts` in 3.12+ new syntax
- Python 3.12+ new style: `def f[*Ts](*args: *Ts) -> tuple[*Ts]`
- `TypedDict` + `Unpack` — type `**kwargs` with a `TypedDict` shape
- Classic use case: `numpy` shape typing, `zip` type correctness, `map` over tuples

## When to Use
- Functions with heterogeneous `*args` where each arg has a distinct type
- Type-safe `zip`, `map`, `starmap` implementations
- **kwargs typed as a TypedDict shape
- Shape-typed arrays (if you're using a framework that supports it)

## When NOT to Use
- Homogeneous `*args` (use `*args: int` or `*args: T`)
- When `tuple[int, str, float]` (fixed size) is sufficient — no need for variadic
- Library code that mypy doesn't yet fully support for variadic generics (check compatibility)

## Examples

```python
from typing import TypeVarTuple, Unpack
from typing import TypedDict

Ts = TypeVarTuple("Ts")

# ✅ Basic variadic — function preserves each arg's type
def identity[*Ts](*args: *Ts) -> tuple[*Ts]:
    return args

result = identity(1, "hello", 3.14)
# result: tuple[int, str, float]  ✅

# ✅ zip-style
def zip_typed[*Ts](
    *iterables: *tuple[list[t] for t in Ts]  # conceptual — actual mypy support varies
) -> list[tuple[*Ts]]: ...

# ✅ Unpack with TypedDict for **kwargs
class Options(TypedDict, total=False):
    timeout: float
    retries: int
    verbose: bool

def fetch(url: str, **kwargs: Unpack[Options]) -> bytes:
    timeout = kwargs.get("timeout", 30.0)
    retries = kwargs.get("retries", 3)
    ...
    return b""

fetch("https://api.example.com", timeout=10.0, retries=5)  # ✅
# fetch("https://...", invalid=True)  # ❌ mypy error: unexpected kwarg

# ✅ 3.12+ new style
def apply[*Ts, R](func: Callable[[*Ts], R], *args: *Ts) -> R:
    return func(*args)

def add(x: int, y: int) -> int:
    return x + y

apply(add, 1, 2)       # ✅ type-checked: apply(Callable[[int, int], int], int, int)
# apply(add, "a", 2)   # ❌ mypy: str is not int
```

## Related Concepts
- [[TypeVar and Generics]]
- [[ParamSpec and Concatenate]]
- [[TypedDict]]

## References
- [PEP 646 – Variadic Generics](https://peps.python.org/pep-0646/)
- [Python docs: `TypeVarTuple`](https://docs.python.org/3/library/typing.html#typing.TypeVarTuple)
- [mypy: Variadic generics](https://mypy.readthedocs.io/en/stable/generics.html#declaring-variadics)
