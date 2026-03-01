---
type: moc
tags: [moc, python, typing, mypy, type-system]
created: 2026-03-01
updated: 2026-03-01
---

# Python Modern Typing MOC

## Overview
Modern Python 3.12+ typing provides a comprehensive, expressive static type system that enables full type safety when used with strict mypy or Pyright. It covers generics, protocols, narrowing, variadic types, and powerful type-level utilities — letting you express precise program invariants without sacrificing readability.

**Prerequisites**: [[Python Basics]], [[OOP in Python]]

---

## Core Concepts
The foundational tools for precise type annotation.

- [[Type Aliases]] — `type` statement (3.12+), `TypeAlias`, naming complex types
- [[TypeVar and Generics]] — `TypeVar`, `Generic[T]`, bounded/constrained vars, `type X[T]` syntax (3.12)
- [[Protocol]] — Structural subtyping, `@runtime_checkable`, duck-typed interfaces
- [[TypedDict]] — Typed dictionaries, `total=False`, `Required`/`NotRequired`, inheritance
- [[Literal Types]] — Exact value constraints, narrowing with literals
- [[Final]] — Constants, `Final[T]`, `@final` on classes/methods
- [[Annotated]] — Metadata layering with `Annotated[T, metadata]`, validator integration
- [[Self Type]] — `Self` for fluent/builder APIs, correct subclass return types
- [[Never and NoReturn]] — Exhaustiveness checking, unreachable branches, `assert_never`
- [[LiteralString]] — SQL injection prevention, taint tracking for string-only APIs
- [[Override]] — `@override` decorator for safe method overriding

---

## Implementation Patterns
How these concepts are applied in real-world code.

### Generic & Structural Patterns
- [[Generic Repository Pattern]] — Type-safe data access with `Generic[T]`
- [[Discriminated Union Pattern]] — `Literal` + `TypedDict` for exhaustive matching
- [[Variance Patterns]] — Covariance/contravariance with `TypeVar(covariant=True)`

### Advanced Typing Patterns
- [[Callable Typing Patterns]] — `Callable`, `ParamSpec`, `Concatenate` for decorator typing
- [[Builder Pattern with TypedDict]] — Step-by-step typed construction
- [[Runtime Narrowing Pattern]] — `TypeGuard`, `TypeIs`, `isinstance`, `assert_never`

### Tooling & Configuration
- [[Mypy Strict Mode Config]] — `mypy.ini` / `pyproject.toml` config, common overrides

---

## Advanced Topics
For deeper understanding.

- [[ParamSpec and Concatenate]] — Preserving callable signatures through decorators, `P.args`/`P.kwargs`
- [[TypeGuard and TypeIs]] — User-defined type narrowing functions
- [[Unpack and TypeVarTuple]] — Variadic generics for tuple-typed args (`*args: *Ts`)
- [[Required and NotRequired]] — Fine-grained `TypedDict` field optionality

---

## Common Problems & Solutions
- How to type a decorator that preserves the wrapped function's signature? → [[Callable Typing Patterns]]
- How to type a mixin or abstract interface without inheritance? → [[Protocol]]
- How to type a `dict` with known, heterogeneous keys? → [[TypedDict]]
- How to make an exhaustive `match`/`if-elif` chain fail at type-check if a case is missed? → [[Never and NoReturn]]
- How to type `*args` of heterogeneous types? → [[Unpack and TypeVarTuple]]
- How to annotate a method that returns `Self` in subclasses? → [[Self Type]]
- How to prevent runtime errors from dynamic SQL strings? → [[LiteralString]]

---

## Related Areas
- [[Python Async MOC]]
- [[Python Dataclasses MOC]]
- [[Python OOP MOC]]

---

## Learning Path
Recommended sequence for mastering modern Python typing.

1. Start with [[Type Aliases]] and [[TypeVar and Generics]] for foundational knowledge
2. Learn [[Protocol]] and [[TypedDict]] for structural data modeling
3. Study [[Literal Types]], [[Final]], and [[Annotated]] for value-level precision
4. Master [[Callable Typing Patterns]] and [[ParamSpec and Concatenate]] for decorator typing
5. Explore [[Discriminated Union Pattern]] and [[Runtime Narrowing Pattern]] for control flow typing
6. Level up with [[Unpack and TypeVarTuple]], [[TypeGuard and TypeIs]]
7. Configure [[Mypy Strict Mode Config]] to enforce everything

---

## References
- [Python `typing` module docs](https://docs.python.org/3/library/typing.html)
- [mypy documentation](https://mypy.readthedocs.io/en/stable/)
- [PEP 695 – Type Parameter Syntax](https://peps.python.org/pep-0695/) (Python 3.12)
- [PEP 673 – `Self` Type](https://peps.python.org/pep-0673/)
- [PEP 675 – `LiteralString`](https://peps.python.org/pep-0675/)
- [PEP 681 – `@override`](https://peps.python.org/pep-0681/)
- [PEP 646 – Variadic Generics](https://peps.python.org/pep-0646/)
- [Pyright strict mode](https://github.com/microsoft/pyright/blob/main/docs/configuration.md)
