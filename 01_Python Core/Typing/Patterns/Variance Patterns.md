---
type: pattern
category: code-pattern
tags: [python, typing, variance, covariant, contravariant, mypy]
created: 2026-03-01
modified: 2026-03-01
complexity: advanced
---

# Variance Patterns

## Problem
When using generics, the direction of subtype relationships can be confusing. A `list[Dog]` is NOT a `list[Animal]` even if `Dog` is an `Animal`. Getting variance wrong leads to mypy errors or (worse) silent type unsafety.

## Solution
Understand and apply covariance (output-only), contravariance (input-only), and invariance (read+write) correctly. Python 3.12 provides clean syntax for declaring variance in the new type parameter syntax.

## Structure

```python
# Invariant — default, for read+write containers
class Container[T]: ...

# Covariant — T only appears in output positions (return types)
class Producer[T_co: covariant]: ...   # 3.12+ syntax
# Old: T_co = TypeVar("T_co", covariant=True)

# Contravariant — T only appears in input positions (parameter types)
class Consumer[T_contra: contravariant]: ...  # 3.12+ syntax
# Old: T_contra = TypeVar("T_contra", contravariant=True)
```

## Full Example

```python
from typing import TypeVar

# ✅ Invariant — default for mutable containers
# list[Dog] is NOT a subtype of list[Animal] — you could append a Cat!
class MutableBox[T]:
    def __init__(self, value: T) -> None:
        self._value = value

    def get(self) -> T:
        return self._value

    def set(self, value: T) -> None:  # write → invariant is necessary
        self._value = value

# ✅ Covariant — read-only (producer), "T_co" convention
# ImmutableBox[Dog] IS a subtype of ImmutableBox[Animal]
class ImmutableBox[T_co: covariant]:
    def __init__(self, value: T_co) -> None:
        self._value = value

    def get(self) -> T_co:  # Only in output position ✅
        return self._value

class Animal: ...
class Dog(Animal): ...

def show_animal(box: ImmutableBox[Animal]) -> None:
    print(box.get())

dog_box: ImmutableBox[Dog] = ImmutableBox(Dog())
show_animal(dog_box)   # ✅ covariance: Dog is Animal, so ImmutableBox[Dog] is ImmutableBox[Animal]

# ✅ Contravariant — write-only (consumer), "T_contra" convention
# Sink[Animal] IS a subtype of Sink[Dog] — backwards!
class Sink[T_contra: contravariant]:
    def consume(self, value: T_contra) -> None:  # Only in input position ✅
        print(f"Consumed: {value!r}")

def feed_dog(sink: Sink[Dog]) -> None:
    sink.consume(Dog())

animal_sink: Sink[Animal] = Sink()
feed_dog(animal_sink)  # ✅ contravariance: Sink[Animal] accepts Dog → is Sink[Dog]

# ✅ Real-world: Callable variance
# Callable is covariant in return type, contravariant in parameter types
type AnimalFactory = "Callable[[], Animal]"
type DogFactory = "Callable[[], Dog]"
# DogFactory is subtype of AnimalFactory (covariant return)

type DogSink = "Callable[[Dog], None]"
type AnimalSink = "Callable[[Animal], None]"
# AnimalSink is subtype of DogSink (contravariant param)

# ✅ Practical: Iterator (covariant in T)
# Iterator[Dog] IS-A Iterator[Animal] — you only get values, never put them in
from collections.abc import Iterator, Generator

def produce_dogs() -> Iterator[Dog]:
    yield Dog()
    yield Dog()

def consume_animals(animals: Iterator[Animal]) -> None:
    for a in animals:
        print(a)

consume_animals(produce_dogs())  # ✅ Iterator is covariant

# ✅ Protocol variance (explicit declaration)
from typing import Protocol

class ReadableStorage[T_co: covariant](Protocol):
    def read(self) -> T_co: ...

class WritableStorage[T_contra: contravariant](Protocol):
    def write(self, value: T_contra) -> None: ...
```

## Links and resources
- [[TypeVar and Generics]]
- [[Protocol]]
- [mypy: Variance](https://mypy.readthedocs.io/en/stable/generics.html#variance-of-generic-types)
- [PEP 695 – Type Parameter Syntax](https://peps.python.org/pep-0695/)
