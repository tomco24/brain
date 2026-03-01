---
type: concept
tags: [python, typing, literalstring, security, mypy]
created: 2026-03-01
modified: 2026-03-01
status: evergreen
---

# LiteralString

## Overview
`LiteralString` represents any string that is a **compile-time literal** — either a literal like `"hello"` or a concatenation of literals. It provides static protection against SQL injection, shell injection, and similar taint vulnerabilities by restricting APIs to only accept literal (non-dynamic) strings.

## Key Points
- Available in `typing` since 3.11 (`from typing import LiteralString`)
- Any `str` literal satisfies `LiteralString`; computed strings (variables, `.format()`, f-strings with variables) do NOT
- Concatenation of `LiteralString + LiteralString` → `LiteralString`
- `LiteralString` is a subtype of `str` — you can use a `LiteralString` anywhere `str` is expected
- Type checkers track "taint" — if a user-controlled string touches a `LiteralString` parameter, mypy errors
- Works with f-strings: `f"SELECT * FROM {table}"` is NOT `LiteralString` because `table` is a variable

## When to Use
- `execute(query: LiteralString)` — DB query APIs to prevent SQL injection
- `subprocess.run([cmd: LiteralString])` — shell command safety
- Template systems where only hardcoded strings are acceptable
- Any API that must reject dynamic, user-controlled strings

## When NOT to Use
- General-purpose string functions — too restrictive
- When you need to accept computed strings (use plain `str`)
- Don't use it as a runtime validator — it's purely static

## Examples

```python
from typing import LiteralString
import sqlite3

# ✅ Safe query execution — only accepts hardcoded SQL
def execute_query(conn: sqlite3.Connection, query: LiteralString) -> list[tuple[object, ...]]:
    cursor = conn.execute(query)
    return cursor.fetchall()

execute_query(conn, "SELECT * FROM users")                  # ✅ literal
execute_query(conn, "SELECT * FROM " + "users")            # ✅ literal + literal

user_table = input("Enter table name: ")
# execute_query(conn, f"SELECT * FROM {user_table}")        # ❌ mypy: not LiteralString
# execute_query(conn, "SELECT * FROM " + user_table)        # ❌ mypy: str, not LiteralString

# ✅ Parametrized queries still safe (use ? placeholders)
def safe_fetch(conn: sqlite3.Connection, user_id: int) -> list[tuple[object, ...]]:
    # This is the RIGHT approach for user input:
    return execute_query(conn, "SELECT * FROM users WHERE id = ?")  # ✅

# ✅ Building query libraries with LiteralString
class QueryBuilder:
    def __init__(self, base: LiteralString) -> None:
        self._query: LiteralString = base

    def and_where(self, condition: LiteralString) -> "QueryBuilder":
        self._query = self._query + " AND " + condition  # ✅ still LiteralString
        return self

    def build(self) -> LiteralString:
        return self._query

q = QueryBuilder("SELECT * FROM orders").and_where("status = 'active'").build()
execute_query(conn, q)  # ✅

# ✅ subprocess safety
import subprocess

def run_tool(tool: LiteralString) -> None:
    subprocess.run([tool], check=True)

run_tool("git")         # ✅
run_tool("ls")          # ✅
# user_cmd = input()
# run_tool(user_cmd)    # ❌ mypy: str is not LiteralString
```

## Related Concepts
- [[Literal Types]]
- [[Annotated]]
- [[Final]]

## References
- [PEP 675 – `LiteralString`](https://peps.python.org/pep-0675/)
- [Python docs: `LiteralString`](https://docs.python.org/3/library/typing.html#typing.LiteralString)
- [mypy: LiteralString](https://mypy.readthedocs.io/en/stable/literal_types.html#literalstring)
