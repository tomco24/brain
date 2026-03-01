---
type: concept
tags: [python, typing, annotated, metadata, mypy, validation]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# Annotated

## Overview
`Annotated[T, metadata...]` attaches arbitrary metadata to a type without changing its type-checker meaning. The type remains `T` for mypy/Pyright, but runtime libraries (Pydantic, FastAPI, attrs) can inspect the metadata for validation, documentation, or serialization.

## Key Points
- First argument is the base type: `Annotated[int, ...]` is still `int` to type checkers
- Additional arguments are **free-form metadata** — any object
- Type checkers (mypy, Pyright) **ignore** the metadata — only runtime frameworks use it
- Can be used with `type` alias statement: `type Age = Annotated[int, Ge(0), Le(150)]`
- Pydantic v2 uses `Annotated` as the primary way to attach field validators
- FastAPI uses it for dependency injection: `Annotated[Session, Depends(get_db)]`
- Multiple layers of `Annotated` flatten: `Annotated[Annotated[int, Ge(0)], Le(10)]` == `Annotated[int, Ge(0), Le(10)]`

## When to Use
- Pydantic v2 field constraints: `Annotated[str, Field(min_length=1, max_length=100)]`
- FastAPI dependency injection and query params: `Annotated[int, Query(ge=0)]`
- Attaching documentation, units, or semantic meaning to primitive types
- Custom validators or serializers without subclassing

## When NOT to Use
- When you need the type checker to enforce the constraint → use `Literal`, `TypeVar` bounds, or `Protocol`
- Overloading it with too many metadata objects makes it unreadable
- Don't use it as a replacement for proper newtypes when nominal typing is needed

## Examples

```python
from typing import Annotated
from pydantic import BaseModel, Field, field_validator
from pydantic.functional_validators import AfterValidator

# ✅ Type aliases with constraints
type PositiveInt = Annotated[int, Field(gt=0)]
type NonEmptyStr = Annotated[str, Field(min_length=1)]
type Email = Annotated[str, Field(pattern=r"^[^@]+@[^@]+\.[^@]+$")]

# ✅ Pydantic v2 model with Annotated fields
class User(BaseModel):
    id: PositiveInt
    name: NonEmptyStr
    email: Email
    age: Annotated[int, Field(ge=0, le=150)]

u = User(id=1, name="Alice", email="alice@example.com", age=30)

# ✅ Custom validator via Annotated
def validate_even(v: int) -> int:
    if v % 2 != 0:
        raise ValueError(f"{v!r} is not even")
    return v

type EvenInt = Annotated[int, AfterValidator(validate_even)]

class Data(BaseModel):
    count: EvenInt

# ✅ FastAPI dependency injection
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session

def get_db() -> Session: ...

app = FastAPI()

@app.get("/users")
async def list_users(db: Annotated[Session, Depends(get_db)]) -> list[dict[str, str]]:
    return []

# ✅ Custom metadata for documentation
class Doc:
    def __init__(self, text: str) -> None:
        self.text = text

type Temperature = Annotated[float, Doc("Temperature in Celsius")]
```

## Related Concepts
- [[TypedDict]]
- [[Literal Types]]
- [[Protocol]]

## References
- [PEP 593 – `Annotated`](https://peps.python.org/pep-0593/)
- [Python docs: `Annotated`](https://docs.python.org/3/library/typing.html#typing.Annotated)
- [Pydantic v2: Annotated validators](https://docs.pydantic.dev/latest/concepts/validators/)
- [FastAPI: Annotated](https://fastapi.tiangolo.com/tutorial/query-params-str-validations/)
