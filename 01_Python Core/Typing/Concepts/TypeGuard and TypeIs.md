---
type: concept
tags: [python, typing, typeguard, typeis, narrowing, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# TypeGuard and TypeIs

## Overview
`TypeGuard[T]` and `TypeIs[T]` let you define **custom narrowing functions** — predicates that tell the type checker "if this returns `True`, the argument is of type `T`". They bridge runtime `isinstance`-style checks with static type narrowing for complex cases.

## Key Points

### TypeGuard (3.10+)
- Return type `TypeGuard[T]`: if the function returns `True`, the **first argument** is narrowed to `T`
- One-directional: `False` doesn't narrow (the type stays unchanged)
- The narrowed type doesn't have to be a subtype of the original — type checkers trust you
- Useful for validating arbitrary dicts, API responses, JSON blobs

### TypeIs (3.13+ / `typing_extensions`)
- Stricter than `TypeGuard`: `TypeIs[T]` requires `T` to be a subtype of the input type
- **Bidirectional**: `True` → narrows to `T`; `False` → narrows to the complement
- Preferred for simple `isinstance`-like predicates because it's more sound

## When to Use
- Typing custom `isinstance`-like functions for union types
- Validating external data (JSON) into specific shapes
- Guard functions in conditional chains where builtins (`isinstance`, `isinstance`) aren't enough
- `TypeIs` for simple `None`/`not None` wrappers, typed dict checks

## When NOT to Use
- Plain `isinstance` checks — type checkers understand those directly
- When `match` statements or `assert` narrowing is sufficient
- Don't use `TypeGuard` as a substitute for proper validation (it's a type-level promise, not runtime safety)

## Examples

```python
from typing import TypeGuard, TypeIs
# TypeIs requires Python 3.13+ or typing_extensions
# from typing_extensions import TypeIs

# ✅ TypeGuard — validating a raw dict as a typed structure
from typing import Any

class UserData(TypedDict):
    name: str
    age: int

def is_user_data(data: Any) -> TypeGuard[UserData]:
    return (
        isinstance(data, dict)
        and isinstance(data.get("name"), str)
        and isinstance(data.get("age"), int)
    )

def process(raw: object) -> None:
    if is_user_data(raw):
        print(raw["name"].upper())  # ✅ raw is UserData here
        print(raw["age"] + 1)       # ✅

# ✅ TypeGuard for union narrowing
type Animal = Dog | Cat

class Dog:
    def bark(self) -> None: ...

class Cat:
    def meow(self) -> None: ...

def is_dog(animal: Animal) -> TypeGuard[Dog]:
    return isinstance(animal, Dog)

def interact(animal: Animal) -> None:
    if is_dog(animal):
        animal.bark()  # ✅ narrowed to Dog

# ✅ TypeIs — stricter, bidirectional (Python 3.13+)
def is_not_none[T](value: T | None) -> TypeIs[T]:
    return value is not None

items: list[str | None] = ["a", None, "b", None]
# TypeIs enables narrowing with filter:
non_null = [x for x in items if is_not_none(x)]
# non_null: list[str]  ✅

# ✅ TypeIs for isinstance-like predicates
def is_str(x: str | int) -> TypeIs[str]:
    return isinstance(x, str)

def process_value(x: str | int) -> None:
    if is_str(x):
        print(x.upper())    # ✅ str
    else:
        print(x + 1)         # ✅ int — TypeIs narrows both branches
```

## Related Concepts
- [[Never and NoReturn]]
- [[Literal Types]]
- [[Runtime Narrowing Pattern]]
- [[Protocol]]

## References
- [PEP 647 – `TypeGuard`](https://peps.python.org/pep-0647/)
- [PEP 742 – `TypeIs`](https://peps.python.org/pep-0742/)
- [mypy: TypeGuard](https://mypy.readthedocs.io/en/stable/type_narrowing.html)
