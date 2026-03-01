---
type: pattern
category: code-pattern
tags: [python, typing, typeddict, builder, mypy]
created: 2026-03-01
modified: 2026-03-01
complexity: intermediate
---

# Builder Pattern with TypedDict

## Problem
Complex objects with many optional parameters are often constructed with telescoping constructors or mutable builders. Without types, it's easy to create partial objects or forget required fields. TypedDict + a builder class gives type-safe staged construction.

## Solution
Use `TypedDict` to define the final shape of the built object, and a typed builder class to accumulate fields. Use `Required`/`NotRequired` plus a final `build()` method that returns the completed `TypedDict`.

## Structure

```python
from typing import TypedDict, Required, NotRequired, Self

class Config(TypedDict):
    host: str
    port: int
    debug: NotRequired[bool]
    timeout: NotRequired[float]

class ConfigBuilder:
    def set_host(self, host: str) -> Self: ...
    def set_port(self, port: int) -> Self: ...
    def set_debug(self, debug: bool) -> Self: ...
    def build(self) -> Config: ...
```

## Full Example

```python
from typing import TypedDict, NotRequired, Self, Required
from dataclasses import dataclass, field

# --- Typed target ---
class DatabaseConfig(TypedDict):
    host: str                               # Required
    port: int                               # Required
    database: str                           # Required
    username: str                           # Required
    password: str                           # Required
    ssl: NotRequired[bool]                  # Optional
    pool_size: NotRequired[int]             # Optional
    timeout: NotRequired[float]             # Optional

# --- Builder ---
class DatabaseConfigBuilder:
    def __init__(self) -> None:
        self._host: str | None = None
        self._port: int | None = None
        self._database: str | None = None
        self._username: str | None = None
        self._password: str | None = None
        self._ssl: bool = False
        self._pool_size: int = 5
        self._timeout: float = 30.0

    def host(self, value: str) -> Self:
        self._host = value
        return self

    def port(self, value: int) -> Self:
        self._port = value
        return self

    def database(self, value: str) -> Self:
        self._database = value
        return self

    def credentials(self, username: str, password: str) -> Self:
        self._username = username
        self._password = password
        return self

    def ssl(self, enabled: bool = True) -> Self:
        self._ssl = enabled
        return self

    def pool(self, size: int) -> Self:
        self._pool_size = size
        return self

    def timeout(self, seconds: float) -> Self:
        self._timeout = seconds
        return self

    def build(self) -> DatabaseConfig:
        if self._host is None:
            raise ValueError("host is required")
        if self._port is None:
            raise ValueError("port is required")
        if self._database is None:
            raise ValueError("database is required")
        if self._username is None or self._password is None:
            raise ValueError("credentials are required")

        config: DatabaseConfig = {
            "host": self._host,
            "port": self._port,
            "database": self._database,
            "username": self._username,
            "password": self._password,
            "ssl": self._ssl,
            "pool_size": self._pool_size,
            "timeout": self._timeout,
        }
        return config

# --- Usage ---
config = (
    DatabaseConfigBuilder()
    .host("localhost")
    .port(5432)
    .database("mydb")
    .credentials("admin", "secret")
    .ssl()
    .pool(10)
    .build()
)

# ✅ Type-safe: config is DatabaseConfig — all fields typed
print(config["host"])       # str ✅
print(config["pool_size"])  # int ✅

# ✅ Alternative: dataclass builder for more complex scenarios
@dataclass
class RequestBuilder:
    _method: str = "GET"
    _url: str = ""
    _headers: dict[str, str] = field(default_factory=dict)
    _body: bytes | None = None

    def method(self, value: str) -> Self:
        self._method = value
        return self

    def url(self, value: str) -> Self:
        self._url = value
        return self

    def header(self, key: str, value: str) -> Self:
        self._headers[key] = value
        return self

    def json_body(self, data: dict[str, object]) -> Self:
        import json
        self._body = json.dumps(data).encode()
        return self.header("Content-Type", "application/json")
```

## Links and resources
- [[TypedDict]]
- [[Required and NotRequired]]
- [[Self Type]]
- [Python `Self` type PEP 673](https://peps.python.org/pep-0673/)
