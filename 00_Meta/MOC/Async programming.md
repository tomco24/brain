---
type: moc
tags:
  - moc
  - async
created: 2026-02-15
updated: 2026-02-15
---
# Python Async Programming MOC

## Overview
Asynchronous programming in Python enables concurrent I/O operations without threading complexity. Essential for high-performance web services, API clients, and I/O-bound applications.

**Prerequisites**: [[Python Basics]], [[Functions]], [[Decorators]], [[Context Managers]]

## Core Concepts

- [[Event Loop]] - The heart of async execution
- [[Async/Await Syntax]] - How to write async code
- [[Coroutines]] - Async function objects
- [[Tasks vs Futures]] - Concurrent execution primitives
- [[Asyncio Module]] - Standard library for async

## Common Patterns

### Resource Management
- [[Async Context Managers]] - Proper cleanup of async resources
- [[Connection Pooling Async]] - Database/HTTP connection management
- [[Async Generators]] - Streaming data asynchronously

### Coordination
- [[Asyncio Queue]] - Producer-consumer patterns
- [[Asyncio Locks]] - Synchronization primitives
- [[Gathering Tasks]] - Running multiple coroutines concurrently

### Error Handling
- [[Try-Except in Async]] - Exception handling patterns
- [[Timeouts in Asyncio]] - Preventing hung operations
- [[Cancellation]] - Graceful task termination

## Framework Integration

### FastAPI
- [[FastAPI Async Routes]]
- [[FastAPI Background Tasks]]
- [[FastAPI Dependencies Async]]

### Database
- [[SQLAlchemy Async]]
- [[Asyncpg]] - PostgreSQL async driver
- [[Motor]] - MongoDB async driver

## Advanced Topics

- [[Custom Event Loop]]
- [[Async Iterators Protocol]]
- [[Running Sync Code in Async Context]]
- [[Thread Pool Executors]]

## Common Problems

- [[How do I call sync function from async?]]
- [[Why is my async code slower?]]
- [[Debugging deadlocks in asyncio]]
- [[Mixing async/sync code gotchas]]

## Examples

- [[Example - FastAPI with Async Database]]
- [[Example - Async Web Scraper]]
- [[Example - Concurrent API Client]]

## Related Areas

- [[Concurrency MOC]]
- [[FastAPI MOC]]
- [[Performance Optimization MOC]]

## Learning Path

1. **Foundation**: Start with [[Event Loop]] and [[Async/Await Syntax]]
2. **Practice**: Work through [[Example - Simple Async Script]]
3. **Patterns**: Learn [[Async Context Managers]] and [[Gathering Tasks]]
4. **Framework**: Apply to [[FastAPI Async Routes]]
5. **Advanced**: Explore [[Running Sync Code in Async Context]]

## Performance Considerations

- [[When NOT to use async]] - I/O-bound vs CPU-bound
- [[Async overhead]] - Context switching costs
- [[Benchmarking async code]]

## References

- Python asyncio docs
- [[Literature Note - AsyncIO Book]]
- Real Python async tutorials