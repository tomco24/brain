---
type: concept
tags: [python, typing, protocol, structural-subtyping, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Protocol

## Overview
`Protocol` enables **structural subtyping** (duck typing with type safety): a class satisfies a `Protocol` if it has the required attributes and methods — no explicit inheritance needed. This is the typed equivalent of Python's duck typing philosophy.

## Key Points
- Defined in `typing` (3.8+) and `collections.abc` for standard protocols
- A class satisfies a `Protocol` just by having matching methods/attributes — no `class Foo(MyProtocol)` needed
- `@runtime_checkable` — allows `isinstance(obj, MyProtocol)` checks at runtime (only checks method names, not signatures)
- Protocols can be **generic**: `class Repository[T](Protocol): ...`
- Protocols support **inheritance**: `class ReadWriteStream(Readable, Writable, Protocol): ...`
- **`Protocol` itself** must always appear in the base list when defining a protocol
- Methods with `...` (ellipsis) body are abstract; protocols can also have concrete methods

## When to Use
- Defining interfaces for duck-typed code without forcing inheritance
- Library APIs that accept any object with certain capabilities (e.g., `Closeable`, `Hashable`)
- Decoupling layers: define `UserRepository(Protocol)` in domain, implement in infra
- When you want to type-check existing third-party classes that you can't modify

## When NOT to Use
- When you control both sides and nominal typing (ABC inheritance) is clearer
- When you need runtime `isinstance` checks on method *signatures* (only names are checked)
- Don't use for data containers → prefer [[TypedDict]] or dataclasses
- Don't use `@runtime_checkable` in hot paths — it's slow

## Examples

```python
from typing import Protocol, runtime_checkable
from collections.abc import Iterator

# ✅ Basic protocol — structural interface
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:  # No explicit inheritance!
    def draw(self) -> None:
        print("○")

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())  # ✅ mypy: OK

# ✅ Runtime checkable
@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None: ...

import io
assert isinstance(io.StringIO(), Closeable)  # ✅ True at runtime

# ✅ Generic protocol
class Repository[T](Protocol):
    def get(self, id: int) -> T: ...
    def save(self, entity: T) -> None: ...
    def delete(self, id: int) -> None: ...

# ✅ Protocol with callback attribute
class EventHandler(Protocol):
    # Attribute protocols
    on_error: Callable[[Exception], None]

    def handle(self, event: str) -> None: ...

# ✅ Combining protocols
class Readable(Protocol):
    def read(self, n: int = -1) -> bytes: ...

class Writable(Protocol):
    def write(self, data: bytes) -> int: ...

class ReadWritable(Readable, Writable, Protocol): ...

# ✅ Self-referential protocol (3.11+)
from typing import Self

class Comparable(Protocol):
    def __lt__(self, other: Self) -> bool: ...
```

## Related Concepts
- [[TypeVar and Generics]]
- [[TypedDict]]
- [[Self Type]]
- [[Generic Repository Pattern]]

## References
- [PEP 544 – Protocols](https://peps.python.org/pep-0544/)
- [mypy: Protocols](https://mypy.readthedocs.io/en/stable/protocols.html)
- [Python docs: `Protocol`](https://docs.python.org/3/library/typing.html#typing.Protocol)
