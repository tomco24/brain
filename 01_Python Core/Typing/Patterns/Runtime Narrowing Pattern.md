---
type: pattern
category: code-pattern
tags: [python, typing, narrowing, typeguard, isinstance, mypy]
created: 2026-03-01
modified: 2026-03-01
complexity: intermediate
---

# Runtime Narrowing Pattern

## Problem
When handling values of union types, you need to check which variant you have at runtime. Type checkers understand certain built-in narrowing operations but not all custom checks. Getting this wrong means either losing type information or writing unsafe casts.

## Solution
Use the hierarchy of narrowing tools mypy/Pyright understand natively: `isinstance`, `type()` checks, `None` checks, `Literal` comparisons in `match`/`if`, `TypeGuard`, and `TypeIs` — in order of precision and safety.

## Structure

```python
# Narrowing hierarchy (prefer earlier options):
# 1. isinstance / is None checks (natively understood)
# 2. match statement structural patterns
# 3. Literal discriminant checks (if x["kind"] == "a")
# 4. TypeIs (bidirectional, strict) — 3.13+
# 5. TypeGuard (one-directional) — for complex external validation
# 6. assert + cast (last resort)
```

## Full Example

```python
from typing import TypeGuard, assert_never, cast
from typing import TypedDict, Literal

# ✅ 1. isinstance — always preferred for class unions
class Dog:
    def bark(self) -> None: ...

class Cat:
    def meow(self) -> None: ...

def greet(animal: Dog | Cat) -> None:
    if isinstance(animal, Dog):
        animal.bark()   # ✅ Dog
    else:
        animal.meow()   # ✅ Cat

# ✅ 2. None check
def safe_upper(value: str | None) -> str:
    if value is None:
        return ""
    return value.upper()  # ✅ value is str here

# ✅ 3. match structural patterns
def handle_value(val: int | str | list[int]) -> str:
    match val:
        case int(n):
            return f"int: {n}"
        case str(s):
            return f"str: {s}"
        case [*nums]:
            return f"list of {len(nums)} ints"
        case _ as never:
            assert_never(never)

# ✅ 4. Literal discriminant narrowing
class CircleData(TypedDict):
    shape: Literal["circle"]
    radius: float

class RectData(TypedDict):
    shape: Literal["rect"]
    width: float
    height: float

type ShapeData = CircleData | RectData

def area(shape: ShapeData) -> float:
    match shape["shape"]:
        case "circle":
            return 3.14159 * shape["radius"] ** 2   # ✅ CircleData
        case "rect":
            return shape["width"] * shape["height"]  # ✅ RectData
        case _ as never:
            assert_never(never)

# ✅ 5. TypeGuard for complex external validation
from typing import Any

def is_valid_config(data: Any) -> TypeGuard[dict[str, str]]:
    return (
        isinstance(data, dict)
        and all(isinstance(k, str) and isinstance(v, str) for k, v in data.items())
    )

def process_config(raw: Any) -> None:
    if is_valid_config(raw):
        for key, value in raw.items():
            print(f"{key}: {value.upper()}")  # ✅ value is str

# ✅ 6. assert for single-use narrowing
def get_name(data: dict[str, object]) -> str:
    name = data.get("name")
    assert isinstance(name, str), f"Expected str name, got {type(name)}"
    return name.upper()  # ✅ narrowed to str after assert

# ✅ 7. cast — only as last resort (no runtime effect, bypasses type checker)
def unsafe_example(response: object) -> str:
    # Only use cast when you are CERTAIN of the type and can't narrow otherwise
    return cast(str, response)

# ✅ Chained narrowing
def process(value: int | str | None) -> str:
    if value is None:
        return "none"
    if isinstance(value, int):
        return f"int:{value}"
    # ✅ value must be str here — all other cases handled
    return value.lower()
```

## Links and resources
- [[TypeGuard and TypeIs]]
- [[Never and NoReturn]]
- [[Discriminated Union Pattern]]
- [[Literal Types]]
- [mypy: Type narrowing](https://mypy.readthedocs.io/en/stable/type_narrowing.html)
