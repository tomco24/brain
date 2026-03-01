---
type: concept
tags: [bash, exit-code, exit-status]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I get the exit code of a command?

## Overview
Every command returns an exit status (0-255). `$?` captures the exit code of the most recently executed command. 0 means success, non-zero means failure.

## Key Points
- `$?` holds the exit code of the last command
- Check `$?` immediately after the command - it changes with each execution
- Exit code 0 = success, non-zero = failure
- Use `$PIPESTATUS` array for exit codes in a pipeline
- `set -o pipefail` makes pipeline fail if any command fails

## When to Use
- Error handling in scripts
- Conditional execution based on command success
- Debugging failed commands

## When NOT to Use
- Don't use `$?` after `echo` or any other command that overwrites it

## Related Concepts
- [[Bash scripting MOC]]
- [[Error Handling]]

## Examples

```bash
# Basic exit code check
cmd
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed with code $?"
fi

# Direct command check (preferred)
if cmd; then
    echo "Success"
fi

# Using || and &&
cmd && echo "Success" || echo "Failed"

# Store exit code for later use
cmd
exit_code=$?
echo "Exit code was: $exit_code"

# Pipeline exit codes
cmd1 | cmd2 | cmd3
echo "cmd1: ${PIPESTATUS[0]}, cmd2: ${PIPESTATUS[1]}, cmd3: ${PIPESTATUS[2]}"

# With pipefail
set -o pipefail
cmd1 | cmd2
if [ $? -ne 0 ]; then
    echo "Pipeline failed"
fi

# Custom exit codes
exit 0   # Success
exit 1   # General error
exit 2   # Misuse of command
exit 127 # Command not found
```

## References
- GNU Bash Manual: Exit Status
