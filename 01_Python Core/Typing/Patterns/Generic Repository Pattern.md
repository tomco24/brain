---
type: pattern
category: code-pattern
tags: [python, typing, generics, repository, mypy]
created: 2026-03-01
modified: 2026-03-01
complexity: intermediate
---

# Generic Repository Pattern

## Problem
Data access code is often duplicated across entities — each entity gets its own near-identical `get_by_id`, `save`, `delete` methods. Without generics, you lose type safety or must write repetitive boilerplate for every entity type.

## Solution
Define a generic base `Repository[T]` using a `Protocol` or abstract class, then specialize it for each entity. Type checkers verify the concrete implementation satisfies the contract and that callers receive correctly typed results.

## Structure

```python
from typing import Protocol, runtime_checkable
from abc import ABC, abstractmethod

# ✅ Option A: Protocol-based (structural, no inheritance required)
class Repository[T, ID](Protocol):
    def get(self, id: ID) -> T | None: ...
    def get_all(self) -> list[T]: ...
    def save(self, entity: T) -> T: ...
    def delete(self, id: ID) -> None: ...

# ✅ Option B: ABC-based (nominal inheritance required)
class BaseRepository[T, ID](ABC):
    @abstractmethod
    def get(self, id: ID) -> T | None: ...

    @abstractmethod
    def get_all(self) -> list[T]: ...

    @abstractmethod
    def save(self, entity: T) -> T: ...

    @abstractmethod
    def delete(self, id: ID) -> None: ...
```

## Full Example

```python
from dataclasses import dataclass
from typing import Protocol
from abc import ABC, abstractmethod

# --- Domain ---
@dataclass
class User:
    id: int
    name: str
    email: str

@dataclass
class Product:
    id: int
    name: str
    price: float

# --- Repository Protocol ---
class Repository[T, ID](Protocol):
    def get(self, id: ID) -> T | None: ...
    def get_all(self) -> list[T]: ...
    def save(self, entity: T) -> T: ...
    def delete(self, id: ID) -> None: ...

# --- In-memory implementation (testing) ---
class InMemoryRepository[T, ID]:
    def __init__(self, id_getter: "Callable[[T], ID]") -> None:
        self._store: dict[ID, T] = {}
        self._id_getter = id_getter

    def get(self, id: ID) -> T | None:
        return self._store.get(id)

    def get_all(self) -> list[T]:
        return list(self._store.values())

    def save(self, entity: T) -> T:
        self._store[self._id_getter(entity)] = entity
        return entity

    def delete(self, id: ID) -> None:
        self._store.pop(id, None)

# --- Specialized repositories ---
class UserRepository(InMemoryRepository[User, int]):
    def find_by_email(self, email: str) -> User | None:
        return next((u for u in self.get_all() if u.email == email), None)

# --- Type-safe service layer ---
class UserService:
    def __init__(self, repo: Repository[User, int]) -> None:
        self._repo = repo  # Accepts ANY repo satisfying the Protocol

    def register(self, name: str, email: str) -> User:
        user = User(id=hash(email), name=name, email=email)
        return self._repo.save(user)

    def get_user(self, user_id: int) -> User:
        user = self._repo.get(user_id)
        if user is None:
            raise KeyError(f"User {user_id} not found")
        return user

# Usage
from typing import Callable
repo = UserRepository(id_getter=lambda u: u.id)
service = UserService(repo)
user = service.register("Alice", "alice@example.com")
print(service.get_user(user.id))
```

## Links and resources
- [[Protocol]]
- [[TypeVar and Generics]]
- [[Self Type]]
- [mypy: Protocols](https://mypy.readthedocs.io/en/stable/protocols.html)
