---
type: pattern
category: code-pattern
tags: [python, typing, callable, overload, paramspec, mypy]
created: 2026-03-01
modified: 2026-03-01
complexity: intermediate
---

# Callable Typing Patterns

## Problem
Python functions are first-class objects. Typing them precisely — especially as arguments, return values, or in decorator scenarios — requires knowing the right tool for each case: `Callable`, `overload`, `ParamSpec`, and `Concatenate`.

## Solution
Use the right `Callable` form for the situation: bare `Callable[..., T]` for simple cases, precise parameter lists for known signatures, `ParamSpec` for decorators, and `@overload` when the return type depends on an argument's literal value.

## Structure

```python
from collections.abc import Callable  # Preferred over typing.Callable
from typing import ParamSpec, TypeVar, overload

P = ParamSpec("P")
T = TypeVar("T")
R = TypeVar("R")

# Decorator that preserves signature
def decorator(func: Callable[P, T]) -> Callable[P, T]: ...

# Function accepting a callback
def on_event(callback: Callable[[str, int], None]) -> None: ...

# Overloaded return based on input Literal
@overload
def parse(raw: str, *, strict: Literal[True]) -> int: ...
@overload
def parse(raw: str, *, strict: Literal[False]) -> int | None: ...
def parse(raw: str, *, strict: bool = False) -> int | None: ...
```

## Full Example

```python
from collections.abc import Callable, Awaitable
from typing import ParamSpec, TypeVar, overload, Literal
from functools import wraps
import time

P = ParamSpec("P")
T = TypeVar("T")

# ✅ 1. Simple callback types
type Handler = Callable[[str], None]
type AsyncHandler = Callable[[str], Awaitable[None]]
type Predicate[T] = Callable[[T], bool]
type Transform[T, R] = Callable[[T], R]

def filter_items[T](items: list[T], pred: Predicate[T]) -> list[T]:
    return [x for x in items if pred(x)]

result = filter_items([1, 2, 3, 4], lambda x: x % 2 == 0)  # ✅ list[int]

# ✅ 2. Decorator preserving full signature (ParamSpec)
def timeit(func: Callable[P, T]) -> Callable[P, T]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.3f}s")
        return result
    return wrapper

@timeit
def compute(x: int, y: int = 10) -> float:
    return x / y

compute(100, y=5)  # ✅ mypy: (x: int, y: int = ...) -> float preserved

# ✅ 3. Overloaded return type based on Literal argument
@overload
def open_channel(mode: Literal["sync"]) -> "SyncChannel": ...
@overload
def open_channel(mode: Literal["async"]) -> "AsyncChannel": ...
@overload
def open_channel(mode: str) -> "SyncChannel | AsyncChannel": ...

def open_channel(mode: str) -> "SyncChannel | AsyncChannel":
    if mode == "sync":
        return SyncChannel()
    return AsyncChannel()

class SyncChannel:
    def send(self, data: bytes) -> None: ...

class AsyncChannel:
    async def send(self, data: bytes) -> None: ...

sync = open_channel("sync")   # ✅ type: SyncChannel
asyn = open_channel("async")  # ✅ type: AsyncChannel

# ✅ 4. Callable with Concatenate (inject extra arg)
from typing import Concatenate

type WithDB[**P, T] = Callable[Concatenate["DB", P], T]

def with_db[**P, T](func: WithDB[P, T]) -> Callable[P, T]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        db = DB.connect()
        return func(db, *args, **kwargs)
    return wrapper

class DB:
    @classmethod
    def connect(cls) -> "DB": ...

@with_db
def get_user(db: DB, user_id: int) -> str:
    ...

get_user(42)  # ✅ db is injected — caller only provides user_id

# ✅ 5. Higher-order typed functions
def compose[A, B, C](
    f: Callable[[B], C],
    g: Callable[[A], B],
) -> Callable[[A], C]:
    return lambda x: f(g(x))

to_upper: Callable[[str], str] = str.upper
to_list: Callable[[str], list[str]] = list
chars_upper = compose(to_list, to_upper)  # Callable[[str], list[str]]
```

## Links and resources
- [[ParamSpec and Concatenate]]
- [[TypeVar and Generics]]
- [[Unpack and TypeVarTuple]]
- [Python docs: `overload`](https://docs.python.org/3/library/typing.html#typing.overload)
- [mypy: `Callable`](https://mypy.readthedocs.io/en/stable/kinds_of_types.html#callable-types-and-lambdas)
