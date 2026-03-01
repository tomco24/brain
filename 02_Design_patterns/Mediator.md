---
type: pattern
category:
  - mediator
  - design-pattern
tags: []
created: 2026-02-22
modified: 2026-02-22
complexity:
---


# Mediator (Event Bus)

## Problem

Components in a system end up tightly coupled when they communicate directly with each other, creating a hard-to-maintain many-to-many dependency graph. As the system grows, changing one component risks breaking all others it references.sbcode+1

## Solution

Introduce a central **EventBus** object (the mediator) that all components communicate through exclusively. Publishers fire typed events into the bus without knowing who handles them; subscribers register handlers for specific event types without knowing who publishes them. This reduces N×M direct dependencies to N+M bus dependencies.github+2

## Structure

```python
import asyncio
from dataclasses import dataclass
from collections import defaultdict
from typing import Callable, Awaitable

# 1. Events — plain data objects (the "messages")
@dataclass
class UserRegistered:
    user_id: int
    email: str

# 2. Mediator — the EventBus
type Handler[T] = Callable[[T], Awaitable[None]]

class EventBus:
    def __init__(self) -> None:
        self._handlers: dict[type, list[Handler]] = defaultdict(list)

    def subscribe[T](self, event_type: type[T], handler: Handler[T]) -> None:
        self._handlers[event_type].append(handler)

    async def publish[T](self, event: T) -> None:
        await asyncio.gather(
            *(h(event) for h in self._handlers.get(type(event), []))
        )

# 3. Subscribers (colleagues) — only depend on the bus + event types
class EmailService:
    async def on_user_registered(self, event: UserRegistered) -> None:
        print(f"Welcome email → {event.email}")

# 4. Wiring (composition root)
async def main() -> None:
    bus = EventBus()
    email = EmailService()

    bus.subscribe(UserRegistered, email.on_user_registered)
    await bus.publish(UserRegistered(user_id=1, email="alice@example.com"))

asyncio.run(main())

```

## Links and Resources

|Resource|Type|Description|
|---|---|---|
|[Refactoring.Guru — Mediator](https://refactoring.guru/design-patterns/mediator) [[refactoring](https://refactoring.guru/design-patterns/mediator)]​|Article|Canonical pattern explanation with UML, pros/cons, and relations to other patterns|
|[Refactoring.Guru — Mediator in Python](https://refactoring.guru/design-patterns/mediator/python/example) [[refactoring](https://refactoring.guru/design-patterns/mediator/python/example)]​|Code example|Full Python code example with detailed comments|
|[sbcode.net — Mediator in Python](https://sbcode.net/python/mediator/) [[sbcode](https://sbcode.net/python/mediator/)]​|Tutorial|Practical Python walkthrough with summary and comparisons to Facade|
|[github.com/dlski/python-mediator](https://github.com/dlski/python-mediator) [[github](https://github.com/dlski/python-mediator)]​|Library|Async CQRS + Event Sourcing micro-framework; auto-discovers handlers by type inspection|
|[github.com/akhundMurad/diator](https://github.com/akhundMurad/diator) [[github](https://github.com/akhundMurad/diator)]​|Library|Python CQRS library built on the Mediator pattern; integrates well with clean architecture|
|[dev.to — Implementing CQRS in Python](https://dev.to/akhundmurad/implementing-cqrs-in-python-41aj) [[dev](https://dev.to/akhundmurad/implementing-cqrs-in-python-41aj)]​|Article|Walkthrough of CQRS using a mediator bus with Commands, Queries, and Events|
|[SourceMaking — Mediator](https://sourcemaking.com/design_patterns/mediator) [[sourcemaking](https://sourcemaking.com/design_patterns/mediator)]​|Reference|GoF-aligned explanation of the pattern with intent, motivation, and applicability|