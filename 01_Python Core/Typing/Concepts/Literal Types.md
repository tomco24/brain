---
type: concept
tags: [python, typing, literal, narrowing, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Literal Types

## Overview
`Literal[...]` restricts a type to exact, specific values rather than just a type category. It's the backbone of discriminated unions, API contracts that accept only certain strings/ints, and precise type narrowing.

## Key Points
- Accepts any combination of: `str`, `int`, `float`, `bool`, `bytes`, `None`, and enum values
- Unions of `Literal` collapse: `Literal["a"] | Literal["b"]` == `Literal["a", "b"]`
- Type checkers narrow `Literal` in `if`/`match` branches automatically
- Functions can be **overloaded** based on `Literal` arguments to return different types
- `Literal` values at runtime are still just plain Python values — no overhead

## When to Use
- Function arguments that accept a fixed set of string/int options (avoid using `str` then checking at runtime)
- Discriminant fields in [[Discriminated Union Pattern]]
- `@overload` signatures where the return type depends on an exact argument value
- Config/enum-like constants without a full `Enum` class

## When NOT to Use
- Large sets of values → use `Enum` (more readable, iterable)
- When values aren't known at type-check time → plain `str`/`int`
- Don't use `Literal` to replace validation — it only checks statically

## Examples

```python
from typing import Literal, overload

# ✅ Basic — only "read" or "write" allowed
type Mode = Literal["read", "write", "append"]

def open_file(path: str, mode: Mode) -> None:
    ...

open_file("data.csv", "read")   # ✅
# open_file("data.csv", "exec") # ❌ mypy error

# ✅ Discriminated union with Literal
class SuccessResponse(TypedDict):
    status: Literal["success"]
    data: str

class ErrorResponse(TypedDict):
    status: Literal["error"]
    message: str

type APIResponse = SuccessResponse | ErrorResponse

def handle(response: APIResponse) -> None:
    if response["status"] == "success":
        print(response["data"])   # ✅ mypy knows it's SuccessResponse
    else:
        print(response["message"])  # ✅ mypy knows it's ErrorResponse

# ✅ Overload based on literal argument
@overload
def get_value(key: Literal["name"]) -> str: ...
@overload
def get_value(key: Literal["age"]) -> int: ...
@overload
def get_value(key: str) -> str | int: ...

def get_value(key: str) -> str | int:
    data: dict[str, str | int] = {"name": "Alice", "age": 30}
    return data[key]

name: str = get_value("name")   # ✅ inferred as str
age: int = get_value("age")     # ✅ inferred as int

# ✅ Literal True / False
def assert_condition(value: bool, *, strict: Literal[True]) -> None: ...
```

## Related Concepts
- [[TypedDict]]
- [[Never and NoReturn]]
- [[TypeGuard and TypeIs]]
- [[Discriminated Union Pattern]]

## References
- [PEP 586 – Literal Types](https://peps.python.org/pep-0586/)
- [mypy: Literal types](https://mypy.readthedocs.io/en/stable/literal_types.html)
