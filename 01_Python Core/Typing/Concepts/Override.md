---
type: concept
tags: [python, typing, override, inheritance, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Override

## Overview
The `@override` decorator (3.12+, or `typing_extensions`) marks a method as intentionally overriding a base class method. Type checkers error if the base class has no matching method — preventing silent bugs when a base class method is renamed or removed.

## Key Points
- `from typing import override` (3.12+) or `from typing_extensions import override`
- mypy in strict mode: `--warn-unused-ignores` + `--enable-error-code=override` catches override mismatches
- **No runtime effect** — purely a type-checking marker
- Works on regular methods, class methods, static methods, and properties
- Catches: typo in method name, base class method renamed, method removed from base class
- Should be used alongside `@abstractmethod` / `Protocol` methods

## When to Use
- Any method that overrides a base class method — make the intent explicit
- Adding `@override` to existing codebases incrementally to guard against future renames
- Large inheritance hierarchies where silently losing an override is a real risk

## When NOT to Use
- Methods that **add** new behavior (not overriding anything) — they don't need `@override`
- Don't add `@override` to `Protocol` implementations where structural typing is used (it's optional but valid)

## Examples

```python
from typing import override

class Animal:
    def speak(self) -> str:
        return "..."

    def move(self) -> str:
        return "Moving"

class Dog(Animal):
    @override
    def speak(self) -> str:     # ✅ OK — Animal.speak exists
        return "Woof!"

    # @override
    # def spake(self) -> str:   # ❌ mypy error: Method "spake" is marked as an override,
    #     return "typo!"        #    but no base class has a matching method

# ✅ With abstract base
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

class Circle(Shape):
    def __init__(self, radius: float) -> None:
        self.radius = radius

    @override
    def area(self) -> float:
        import math
        return math.pi * self.radius ** 2

    @override
    def perimeter(self) -> float:
        import math
        return 2 * math.pi * self.radius

# ✅ Catching rename bugs
class DataProcessor:
    def process(self, data: list[int]) -> list[int]:
        return sorted(data)

class FilterProcessor(DataProcessor):
    @override
    def process(self, data: list[int]) -> list[int]:   # ✅ Safe
        return [x for x in data if x > 0]

# If DataProcessor.process is renamed to .run:
# @override
# def process → ❌ mypy: no base method 'process' — caught!

# ✅ Works with properties and classmethods
class Base:
    @property
    def name(self) -> str:
        return "base"

    @classmethod
    def create(cls) -> "Base":
        return cls()

class Child(Base):
    @property
    @override
    def name(self) -> str:
        return "child"

    @classmethod
    @override
    def create(cls) -> "Child":
        return cls()
```

## Related Concepts
- [[Self Type]]
- [[Protocol]]
- [[Final]]

## References
- [PEP 698 – `@override`](https://peps.python.org/pep-0698/)
- [Python docs: `override`](https://docs.python.org/3/library/typing.html#typing.override)
- [mypy: `@override`](https://mypy.readthedocs.io/en/stable/error_code_list.html#check-that-override-is-compatible-override)
