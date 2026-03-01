---
type: concept
tags: [python, typing, never, noreturn, exhaustiveness, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Never and NoReturn

## Overview
`Never` and `NoReturn` represent the **bottom type** — a type with no values. They signal that a function never returns normally (raises always, infinite loop) or that a code path is unreachable. Combined with `assert_never`, they enable **exhaustiveness checking** at type-check time.

## Key Points
- `NoReturn`: return annotation for functions that never return (raise or loop forever) — available since 3.6
- `Never` (3.11+): the explicit bottom type usable in any position, not just return annotations — identical to `NoReturn` conceptually but more general
- `assert_never(x: Never)` — a helper that fails at type-check time if `x` isn't narrowed to `Never`; fails at runtime with `AssertionError`
- If all branches of a union are handled, mypy infers the remainder as `Never`
- `Never` is a subtype of **everything** — it can be assigned to any type (vacuously true)

## When to Use
- Functions that always raise: `def fail(msg: str) -> NoReturn: raise RuntimeError(msg)`
- `assert_never` at the end of `match`/`if-elif` ladders to get exhaustiveness guarantees
- Generic code that should be impossible to instantiate in certain branches

## When NOT to Use
- Optional functions that sometimes return `None` → use `T | None`
- Try/except blocks — exceptions don't make a function `NoReturn` unless every path raises

## Examples

```python
from typing import Never, NoReturn, assert_never

# ✅ Function that always raises
def unreachable(msg: str = "unreachable") -> NoReturn:
    raise AssertionError(msg)

def fail(msg: str) -> NoReturn:
    raise RuntimeError(msg)

# ✅ Exhaustiveness checking with assert_never
from typing import Literal

type Direction = Literal["north", "south", "east", "west"]

def move(direction: Direction) -> str:
    match direction:
        case "north":
            return "Moving north"
        case "south":
            return "Moving south"
        case "east":
            return "Moving east"
        case "west":
            return "Moving west"
        case _ as unreachable_direction:
            assert_never(unreachable_direction)
            # ✅ If you add "up" to Direction and forget to add a case,
            # mypy will error here: Argument 1 to "assert_never" has
            # incompatible type "Literal['up']"; expected "Never"

# ✅ With TypedDict discriminated union
class Dog(TypedDict):
    kind: Literal["dog"]
    name: str

class Cat(TypedDict):
    kind: Literal["cat"]
    indoor: bool

type Animal = Dog | Cat

def describe(animal: Animal) -> str:
    match animal["kind"]:
        case "dog":
            return f"Dog: {animal['name']}"
        case "cat":
            return f"Cat, indoor: {animal['indoor']}"
        case _ as never:
            assert_never(never)  # ✅ Exhaustive — mypy confirms no cases missed

# ✅ Never in generic code
def first_or_raise[T](lst: list[T]) -> T:
    if not lst:
        fail("Empty list")  # NoReturn — mypy knows this branch exits
    return lst[0]  # ✅ lst[0] is typed as T, not T | Never
```

## Related Concepts
- [[Literal Types]]
- [[TypeGuard and TypeIs]]
- [[Discriminated Union Pattern]]
- [[Runtime Narrowing Pattern]]

## References
- [PEP 544 – `Never`](https://peps.python.org/pep-0544/)
- [Python 3.11: `Never` and `assert_never`](https://docs.python.org/3/library/typing.html#typing.Never)
- [mypy: `NoReturn`](https://mypy.readthedocs.io/en/stable/more_types.html#noreturn)
