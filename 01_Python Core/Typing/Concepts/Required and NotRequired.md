---
type: concept
tags: [python, typing, required, notrequired, typeddict, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Required and NotRequired

## Overview
`Required[T]` and `NotRequired[T]` provide **per-field optionality** for `TypedDict`, overriding the class-level `total=` setting. They enable mixed schemas where some fields are always required and others are optional — without splitting into multiple TypedDicts.

## Key Points
- Available in `typing` since 3.11 (`from typing import Required, NotRequired`)
- `total=True` (default) — all fields required; use `NotRequired[T]` to opt specific fields out
- `total=False` — all fields optional; use `Required[T]` to make specific fields mandatory
- **Inheritance** also respects these markers — a subclass can add its own mix
- At runtime, `Required`/`NotRequired` are just type annotations — no effect on dict behavior
- Replaces the ugly pattern of splitting into two TypedDicts and inheriting

## When to Use
- HTTP API response/request shapes: `id` always present, `updated_at` optional
- Configuration objects: some keys always needed, others have defaults
- Database row types: nullable columns as `NotRequired`
- Any `TypedDict` that was previously split into `FooRequired` + `FooOptional`

## When NOT to Use
- When `total=False` or `total=True` covers all fields uniformly — simpler
- When you need runtime validation of required fields → use Pydantic instead
- Deeply nested mixed schemas → consider a dataclass with `field(default=...)` 

## Examples

```python
from typing import TypedDict, Required, NotRequired

# ✅ Base: total=True, some fields optional
class UserProfile(TypedDict):
    id: int                              # Required (default)
    username: str                        # Required (default)
    email: NotRequired[str]              # Optional
    avatar_url: NotRequired[str | None]  # Optional, can be None
    bio: NotRequired[str]                # Optional

# Valid usages:
profile1: UserProfile = {"id": 1, "username": "alice"}              # ✅
profile2: UserProfile = {"id": 2, "username": "bob", "email": "b@ex.com"}  # ✅
# profile3: UserProfile = {"id": 3}  # ❌ mypy: missing 'username'

# ✅ Base: total=False, some fields required
class PatchRequest(TypedDict, total=False):
    id: Required[int]       # Always required even in a PATCH
    name: str               # Optional
    email: str              # Optional
    age: int                # Optional

patch: PatchRequest = {"id": 42}                        # ✅
patch2: PatchRequest = {"id": 42, "name": "Alice"}     # ✅
# bad: PatchRequest = {"name": "Alice"}                 # ❌ mypy: missing 'id'

# ✅ Inheritance with mixed totals
class BaseConfig(TypedDict):
    host: str                               # Required
    port: int                               # Required

class ExtendedConfig(BaseConfig, total=False):
    debug: bool                             # Optional (total=False on this class)
    log_level: str                          # Optional
    timeout: Required[float]                # Required despite total=False

config: ExtendedConfig = {
    "host": "localhost",
    "port": 5432,
    "timeout": 30.0,
}  # ✅

# ✅ Reading fields safely with .get()
def get_email(user: UserProfile) -> str:
    return user.get("email", "no-email@example.com")  # ✅ safe access
```

## Related Concepts
- [[TypedDict]]
- [[Annotated]]
- [[Literal Types]]

## References
- [PEP 655 – `Required`/`NotRequired`](https://peps.python.org/pep-0655/)
- [Python docs: `Required`](https://docs.python.org/3/library/typing.html#typing.Required)
- [mypy: TypedDict](https://mypy.readthedocs.io/en/stable/typed_dict.html)
