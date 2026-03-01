---
type: concept
tags: [python, typing, typeddict, mypy, structured-data]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# TypedDict

## Overview
`TypedDict` creates a typed `dict` subtype where specific string keys map to specific value types. It gives type safety to plain `dict` literals — essential for JSON data, config dicts, and inter-module data contracts without the overhead of full classes.

## Key Points
- Keys are **required** by default; use `total=False` to make all optional, or `Required`/`NotRequired` for per-field control (3.11+)
- TypedDicts are **structurally typed** — any dict with the right keys satisfies the type
- Support **inheritance**: `class AdminUser(User): extra_field: str`
- Cannot use `Final` or mutable default values inside `TypedDict`
- **Two syntaxes**: class-based (preferred) and functional `TypedDict("Name", {...})`
- At runtime, a `TypedDict` is just a plain `dict` — no extra overhead
- mypy in strict mode will catch missing required keys, wrong value types, and extra keys

## When to Use
- Typing JSON API responses / request payloads
- Config / options dictionaries with known shapes
- Return types for functions that naturally return dicts (avoid if a dataclass/attrs is cleaner)
- Interop with libraries that return plain dicts (e.g., DB rows, HTTP responses)

## When NOT to Use
- When you need methods or behavior → use `dataclass` or a regular class
- When keys are dynamic / unknown at type-check time → `dict[str, Any]`
- When you need immutability → `dataclass(frozen=True)` or `NamedTuple`
- Very deep nesting — nested TypedDicts can be hard to read; consider Pydantic

## Examples

```python
from typing import TypedDict, Required, NotRequired

# ✅ Basic TypedDict
class Point(TypedDict):
    x: float
    y: float

p: Point = {"x": 1.0, "y": 2.0}  # ✅
# p2: Point = {"x": 1.0}  # ❌ mypy: missing key 'y'

# ✅ Partial TypedDict (all optional)
class Config(TypedDict, total=False):
    debug: bool
    timeout: int
    retries: int

# ✅ Mixed required/optional (3.11+)
class User(TypedDict):
    id: int                         # Required
    name: str                        # Required
    email: NotRequired[str]          # Optional
    role: NotRequired[str]           # Optional

# ✅ Inheritance
class AdminUser(User):
    permissions: list[str]           # Required additional field

# ✅ Nested TypedDict
class Address(TypedDict):
    street: str
    city: str
    country: str

class Person(TypedDict):
    name: str
    age: int
    address: Address

# ✅ Functional syntax (useful when keys are not valid identifiers)
HTTPHeaders = TypedDict("HTTPHeaders", {
    "Content-Type": str,
    "X-Request-Id": str,
})

# ✅ TypedDict as return type of a JSON-parsing function
import json

def parse_config(raw: str) -> Config:
    data = json.loads(raw)
    return data  # mypy will check this assignment

# ✅ Combining with Unpack for **kwargs typing (3.12+)
from typing import Unpack

def create_user(**kwargs: Unpack[User]) -> None:
    ...
```

## Related Concepts
- [[Required and NotRequired]]
- [[Protocol]]
- [[Annotated]]
- [[Discriminated Union Pattern]]
- [[Builder Pattern with TypedDict]]

## References
- [PEP 589 – TypedDict](https://peps.python.org/pep-0589/)
- [PEP 655 – `Required`/`NotRequired`](https://peps.python.org/pep-0655/)
- [mypy: TypedDict](https://mypy.readthedocs.io/en/stable/typed_dict.html)
