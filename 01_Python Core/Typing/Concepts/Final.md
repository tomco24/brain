---
type: concept
tags: [python, typing, final, constants, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Final

## Overview
`Final` marks a variable or attribute as a constant that cannot be reassigned. `@final` marks a class or method as non-overridable. Together, they express immutability and closed hierarchies at the type-check level.

## Key Points
- `Final[T]` / `Final` — variable cannot be re-assigned after initial binding
- `@final` on a **class** — prevents subclassing (enforced by mypy, not Python runtime)
- `@final` on a **method** — prevents overriding in subclasses
- `ClassVar[Final[T]]` — class-level constant (rare; use `Final` directly on class vars)
- Works with type inference: `MAX: Final = 100` infers `Final[int]`
- Module-level, class-level, and instance-level `Final` all work; can't use `Final` in function scope for attributes (only at module/class level makes sense semantically)

## When to Use
- Module-level constants: `API_URL: Final = "https://api.example.com"`
- Enum-like class constants: `MAX_RETRIES: Final = 3`
- Signaling closed class hierarchies (sealed types pattern)
- Preventing accidental override of critical base-class methods

## When NOT to Use
- Mutable containers (the reference is final but the content isn't — use `tuple` instead of `list` for true immutability)
- Don't confuse with value immutability — `Final[list[int]]` still allows `.append()`
- Don't overuse `@final` — only seal what you deliberately want closed

## Examples

```python
from typing import Final, final

# ✅ Module constants
MAX_CONNECTIONS: Final = 10
BASE_URL: Final[str] = "https://api.example.com"

# ❌ mypy error — cannot reassign Final
# MAX_CONNECTIONS = 20

# ✅ Class constant
class Config:
    DEBUG: Final = False
    MAX_RETRIES: Final[int] = 3

    def __init__(self) -> None:
        self.name: Final = "my-app"  # Instance final — set once in __init__

# ✅ @final class — cannot be subclassed
@final
class Singleton:
    _instance: "Singleton | None" = None

    @classmethod
    def get_instance(cls) -> "Singleton":
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

# class SubSingleton(Singleton): ...  # ❌ mypy error

# ✅ @final method — cannot be overridden
class Base:
    @final
    def core_logic(self) -> None:
        print("Do not override this")

    def customizable(self) -> None:
        ...

class Child(Base):
    # def core_logic(self) -> None: ...  # ❌ mypy error
    def customizable(self) -> None:      # ✅ OK
        print("Overridden")

# ✅ Sealed / closed hierarchy simulation
@final
class Success[T]:
    def __init__(self, value: T) -> None:
        self.value = value

@final
class Failure[E]:
    def __init__(self, error: E) -> None:
        self.error = error

type Result[T, E] = Success[T] | Failure[E]
```

## Related Concepts
- [[Literal Types]]
- [[TypeVar and Generics]]
- [[Never and NoReturn]]

## References
- [PEP 591 – `Final`](https://peps.python.org/pep-0591/)
- [mypy: Final names](https://mypy.readthedocs.io/en/stable/final_attrs.html)
- [Python docs: `Final`](https://docs.python.org/3/library/typing.html#typing.Final)
