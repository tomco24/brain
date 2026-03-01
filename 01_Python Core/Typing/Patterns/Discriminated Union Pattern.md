---
type: pattern
category: code-pattern
tags: [python, typing, literal, typeddict, union, discriminated, mypy]
created: 2026-03-01
modified: 2026-03-01
complexity: intermediate
---

# Discriminated Union Pattern

## Problem
When a value can be one of several different structured types (e.g., API response variants, event types, state machine states), you need a way to distinguish them and have the type checker narrow automatically when you check which variant you have.

## Solution
Use a `TypedDict` (or dataclass) for each variant with a shared `Literal` discriminant field (e.g., `kind`, `type`, `status`). The type checker narrows the union based on the discriminant's value.

## Structure

```python
from typing import Literal, TypedDict

class VariantA(TypedDict):
    kind: Literal["a"]
    # ... A-specific fields

class VariantB(TypedDict):
    kind: Literal["b"]
    # ... B-specific fields

type Union = VariantA | VariantB

def handle(value: Union) -> None:
    match value["kind"]:
        case "a":
            ...  # value narrowed to VariantA
        case "b":
            ...  # value narrowed to VariantB
        case _ as never:
            from typing import assert_never
            assert_never(never)  # exhaustiveness check
```

## Full Example

```python
from typing import Literal, TypedDict, assert_never

# --- API Response variants ---
class OkResponse(TypedDict):
    status: Literal["ok"]
    data: list[dict[str, object]]
    count: int

class ErrorResponse(TypedDict):
    status: Literal["error"]
    code: int
    message: str

class RedirectResponse(TypedDict):
    status: Literal["redirect"]
    location: str

type APIResponse = OkResponse | ErrorResponse | RedirectResponse

# --- Handler with exhaustive narrowing ---
def handle_response(response: APIResponse) -> str:
    match response["status"]:
        case "ok":
            # ✅ response is OkResponse here
            return f"Got {response['count']} items"
        case "error":
            # ✅ response is ErrorResponse here
            return f"Error {response['code']}: {response['message']}"
        case "redirect":
            # ✅ response is RedirectResponse here
            return f"Redirect to {response['location']}"
        case _ as unreachable:
            assert_never(unreachable)  # ✅ mypy errors if any case is missed

# --- Event system ---
class UserCreatedEvent(TypedDict):
    event: Literal["user.created"]
    user_id: int
    email: str

class OrderPlacedEvent(TypedDict):
    event: Literal["order.placed"]
    order_id: int
    total: float

class PaymentFailedEvent(TypedDict):
    event: Literal["payment.failed"]
    order_id: int
    reason: str

type DomainEvent = UserCreatedEvent | OrderPlacedEvent | PaymentFailedEvent

def dispatch(evt: DomainEvent) -> None:
    match evt["event"]:
        case "user.created":
            send_welcome_email(evt["email"])
        case "order.placed":
            process_order(evt["order_id"])
        case "payment.failed":
            notify_payment_failure(evt["order_id"], evt["reason"])
        case _ as never:
            assert_never(never)

def send_welcome_email(email: str) -> None: ...
def process_order(order_id: int) -> None: ...
def notify_payment_failure(order_id: int, reason: str) -> None: ...

# --- With dataclasses (alternative to TypedDict) ---
from dataclasses import dataclass

@dataclass(frozen=True)
class Loading:
    kind: Literal["loading"] = "loading"

@dataclass(frozen=True)
class Loaded[T]:
    kind: Literal["loaded"] = "loaded"
    data: T = None  # type: ignore[assignment]  # set in __init__

@dataclass(frozen=True)
class Failed:
    kind: Literal["failed"] = "failed"
    error: str = ""

type AsyncState[T] = Loading | Loaded[T] | Failed
```

## Links and resources
- [[Literal Types]]
- [[TypedDict]]
- [[Never and NoReturn]]
- [[TypeGuard and TypeIs]]
- [Python `match` statement](https://docs.python.org/3/reference/compound_stmts.html#the-match-statement)
