---
type: concept
tags: [python, typing, paramspec, concatenate, decorators, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# ParamSpec and Concatenate

## Overview
`ParamSpec` captures the full **parameter specification** (argument names, types, defaults) of a callable, so decorators can be typed without losing the wrapped function's signature. `Concatenate` prepends additional parameters to a `ParamSpec`.

## Key Points
- `P = ParamSpec("P")` — captures `(args, kwargs)` of a function
- A decorator `def decorator(f: Callable[P, T]) -> Callable[P, T]` preserves the full signature of `f` for callers
- `P.args` and `P.kwargs` — used inside the wrapper's `*args`/`**kwargs` to forward calls
- `Concatenate[int, P]` — prepends an extra `int` param before `P` (useful for method-to-function adapters, middleware)
- Python 3.12+: `def decorator[**P, T](f: Callable[P, T]) -> Callable[P, T]` — no explicit `ParamSpec` declaration needed

## When to Use
- Typing decorators that forward to the original function (logging, timing, retry, auth)
- Converting bound methods to standalone callables
- Middleware patterns where extra context is prepended to a handler

## When NOT to Use
- Simple decorators that change the signature — use `Callable[..., T]` (or redesign)
- When you don't care about argument inference at call sites (e.g. internal helpers)

## Examples

```python
from typing import ParamSpec, Concatenate, Callable, TypeVar
from functools import wraps
import time

P = ParamSpec("P")
T = TypeVar("T")

# ✅ Logging decorator — full signature preserved
def log_call(func: Callable[P, T]) -> Callable[P, T]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        print(f"Calling {func.__name__!r}")
        result = func(*args, **kwargs)
        print(f"{func.__name__!r} returned {result!r}")
        return result
    return wrapper

@log_call
def add(x: int, y: int) -> int:
    return x + y

add(1, 2)      # ✅ mypy knows signature is (x: int, y: int) -> int

# ✅ Python 3.12+ syntax — no explicit ParamSpec declaration
def retry[**P, T](times: int) -> Callable[[Callable[P, T]], Callable[P, T]]:
    def decorator(func: Callable[P, T]) -> Callable[P, T]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == times - 1:
                        raise
            raise RuntimeError("unreachable")
        return wrapper
    return decorator

@retry(3)
def fetch(url: str, timeout: float = 5.0) -> bytes:
    ...

# ✅ Concatenate — prepend a connection parameter
type WithConnection[**P, T] = Callable[Concatenate[Connection, P], T]

def inject_connection(func: WithConnection[P, T]) -> Callable[P, T]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        conn = get_connection()
        return func(conn, *args, **kwargs)
    return wrapper

class Connection: ...
def get_connection() -> Connection: ...

@inject_connection
def query(conn: Connection, sql: str) -> list[dict[str, object]]:
    ...

# Caller doesn't need to provide conn — signature is (sql: str) -> list[...]
result = query("SELECT 1")  # ✅
```

## Related Concepts
- [[TypeVar and Generics]]
- [[Callable Typing Patterns]]
- [[Unpack and TypeVarTuple]]

## References
- [PEP 612 – `ParamSpec`](https://peps.python.org/pep-0612/)
- [Python docs: `ParamSpec`](https://docs.python.org/3/library/typing.html#typing.ParamSpec)
- [mypy: `ParamSpec`](https://mypy.readthedocs.io/en/stable/generics.html#paramspec)
