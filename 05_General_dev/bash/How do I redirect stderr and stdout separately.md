---
type: concept
tags: [bash, redirection, stderr, stdout]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I redirect stderr and stdout separately?

## Overview
Bash uses file descriptors: 0=stdin, 1=stdout, 2=stderr. You can redirect each independently to control where output and errors go.

## Key Points
- `>` or `1>` redirects stdout
- `2>` redirects stderr
- `&>` redirects both stdout and stderr (bash 4+)
- `2>&1` redirects stderr to wherever stdout goes
- `/dev/null` discards output

## When to Use
- Suppressing error messages: `cmd 2>/dev/null`
- Logging stdout and stderr to separate files
- Combining streams for piping: `cmd 2>&1 | grep`

## When NOT to Use
- `2>&1` must come AFTER the stdout redirect to work correctly

## Related Concepts
- [[Bash scripting MOC]]
- [[Linux Command Line]]

## Examples

```bash
# Redirect stdout to file, stderr to another
cmd > output.log 2> errors.log

# Redirect both to same file
cmd > all.log 2>&1
cmd &> all.log

# Suppress errors only
cmd 2>/dev/null

# Suppress stdout only
cmd > /dev/null

# Pipe both stdout and stderr
cmd 2>&1 | grep pattern

# Redirect stderr to stdout, then pipe
cmd 2>&1 | less

# Swap stdout and stderr
cmd 3>&1 1>&2 2>&3
```

## References
- GNU Bash Manual: Redirections
