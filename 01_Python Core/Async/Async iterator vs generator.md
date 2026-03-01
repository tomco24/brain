---
type: concept
tags: []
created: 2026-02-22
modified: 2026-02-22
status: draft
---

# AsyncIterator vs AsyncGenerator Typing

## Overview

`AsyncIterator[T]` and `AsyncGenerator[YieldType, SendType]` are both types from `collections.abc` that describe async-iterable objects. `AsyncGenerator` is a concrete subtype of `AsyncIterator` — every generator is an iterator, but not every iterator is a generator.[[bjoernricks.github](https://bjoernricks.github.io/work-shop-wednesday/python/asyncio/cpython/async_generators.html)]​

## Key Points

- **`AsyncIterator[T]` is an interface (protocol):** any object implementing `__aiter__()` and `__anext__()` satisfies it, including custom classes[[realpython](https://realpython.com/python-async-iterators/)]​
    
- **`AsyncGenerator[YieldType, SendType]` is a concrete type:** produced exclusively by `async def` functions that contain a `yield` statement; it also exposes `.asend()`, `.athrow()`, and `.aclose()`peps.python+1
    
- **Inheritance chain:** `AsyncGenerator → AsyncIterator → AsyncIterable`, so `AsyncGenerator` is always a valid `AsyncIterator`, but the reverse is false[[bjoernricks.github](https://bjoernricks.github.io/work-shop-wednesday/python/asyncio/cpython/async_generators.html)]​
    
- **`SendType` is almost always `None`** unless you use `.asend()` to push values back into the generator[[peps.python](https://peps.python.org/pep-0525/)]​
    

## When to Use

- Use **`AsyncIterator[T]`** when declaring the return type of an `@asynccontextmanager`-decorated function — mypy's typeshed stubs for `asynccontextmanager` specifically expect `Callable[..., AsyncIterator[T]]` and transform it into `AbstractAsyncContextManager[T]`[[stackoverflow](https://stackoverflow.com/questions/63125259/what-is-the-proper-way-to-type-hint-the-return-value-of-an-asynccontextmanager)]​
    
- Use **`AsyncIterator[T]`** when writing abstract methods or protocols that only require iteration, without guaranteeing generator-specific methods to the caller[[stackoverflow](https://stackoverflow.com/questions/70266649/python-difference-of-typing-with-asyncgenerator-or-asynciterator)]​
    
- Use **`AsyncGenerator[YieldType, None]`** when annotating the return type of any `async def` function that uses `yield`, so the caller has full access to `.aclose()` and `.athrow()`[[github](https://github.com/python/cpython/issues/112866)]​
    
- Use **`AsyncGenerator[YieldType, SendType]`** when the generator uses `value = yield` and the caller pushes data back via `.asend(value)`[[peps.python](https://peps.python.org/pep-0525/)]​
    

## When NOT to Use

- Do **not** annotate an `@asynccontextmanager` function body as `-> AsyncGenerator[T, None]` as the outer wrapper — mypy will not correctly infer the decorator transformationstackoverflow+1
    
- Do **not** annotate the return of a `yield`-based function as `AsyncIterator[T]` if the caller needs `.aclose()` — mypy will block that call because `AsyncIterator` does not guarantee it exists[[github](https://github.com/python/cpython/issues/112866)]​
    
- Do **not** use `AsyncContextManager[T]` or `-> T` as the return annotation of an `@asynccontextmanager` function body — these are also wrong and will fail under `--strict`[[stackoverflow](https://stackoverflow.com/questions/63125259/what-is-the-proper-way-to-type-hint-the-return-value-of-an-asynccontextmanager)]​
    

## Related Concepts

- `contextlib.asynccontextmanager` — decorator that wraps an `AsyncIterator[T]` function into an `AbstractAsyncContextManager[T]`[[docs.python](https://docs.python.org/3/library/contextlib.html)]​
    
- `AbstractAsyncContextManager[T]` — the actual runtime type of the object returned after `@asynccontextmanager` decoration; what the caller receives[[docs.python](https://docs.python.org/3/library/contextlib.html)]​
    
- `AsyncIterable[T]` — the broadest interface, only requires `__aiter__()`; sits above `AsyncIterator` in the hierarchy[[docs.python](https://docs.python.org/3/library/typing.html)]​
    
- `Generator[YieldType, SendType, ReturnType]` — the synchronous equivalent; note it has a third `ReturnType` parameter (the value of a `return` statement inside the generator)[[docs.python](https://docs.python.org/3/library/typing.html)]​
    

## Examples

``` python
from collections.abc import AsyncGenerator, AsyncIterator
from contextlib import asynccontextmanager

# ✅ AsyncGenerator: concrete yield function
# Caller can call .aclose(), .asend(), .athrow()
async def stream_events(topic: str) -> AsyncGenerator[Event, None]:
    async for raw in source:
        yield parse(raw)

# ✅ @asynccontextmanager: outer wrapper must be AsyncIterator
# Inner yielded value is AsyncGenerator — be precise about the payload
@asynccontextmanager
async def subscribe(topic: str) -> AsyncIterator[AsyncGenerator[Event, None]]:
    queue: asyncio.Queue[Event] = asyncio.Queue(maxsize=512)
    register(topic, queue)
    try:
        yield _drain(queue)   # <-- AsyncGenerator[Event, None]
    finally:
        unregister(topic, queue)

# ❌ Wrong outer wrapper
async def subscribe_wrong(topic: str) -> AsyncGenerator[AsyncIterator[Event], None]: ...

# ❌ Inner typed too broadly — caller loses .aclose() access
async def subscribe_imprecise(topic: str) -> AsyncIterator[AsyncIterator[Event]]: ...

```

## References

- Python docs: `typing` — `AsyncGenerator`, `AsyncIterator`[[docs.python](https://docs.python.org/3/library/typing.html)]​
    
- PEP 525 — Asynchronous Generators[[peps.python](https://peps.python.org/pep-0525/)]​
    
- CPython issue #112866 — `AsyncIterator` vs `AsyncGenerator` typing docs clarification[[github](https://github.com/python/cpython/issues/112866)]​
    
- Stack Overflow: Correct type hints with `AsyncGenerator` and `AsyncContextManager`[[stackoverflow](https://stackoverflow.com/questions/68905848/how-to-correctly-specify-type-hints-with-asyncgenerator-and-asynccontextmanager)]​
    
- Stack Overflow: Proper return type hint for `@asynccontextmanager`[[stackoverflow](https://stackoverflow.com/questions/63125259/what-is-the-proper-way-to-type-hint-the-return-value-of-an-asynccontextmanager)]​
