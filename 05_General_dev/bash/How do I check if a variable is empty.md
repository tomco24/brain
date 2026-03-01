---
type: concept
tags: [bash, variables, empty-check]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I check if a variable is empty?

## Overview
Bash provides several ways to check if a variable is empty or unset: test operators `-z`/`-n`, parameter expansion defaults, and conditional checks.

## Key Points
- `-z "$var"` returns true if string is empty (zero length)
- `-n "$var"` returns true if string is not empty
- `[ "$var" ]` is true if non-empty (equivalent to `-n`)
- `${var:-default}` provides default if empty or unset
- `${var:=default}` sets default if empty or unset
- `${var:+value}` returns value if var is set and non-empty

## When to Use
- Validating required arguments
- Setting default values
- Conditional logic based on variable state

## When NOT to Use
- Always quote `$var` in `[ ]` to prevent errors with empty/unset variables
- Use `[[ ]]` for safer testing without quoting requirement

## Related Concepts
- [[How do I compare strings in bash?]]
- [[Bash scripting MOC]]

## Examples

```bash
# Test if empty
if [ -z "$var" ]; then
    echo "Variable is empty"
fi

# Test if not empty
if [ -n "$var" ]; then
    echo "Variable has content"
fi

# Shorthand (not empty)
if [ "$var" ]; then
    echo "Variable has content"
fi

# Shorthand (empty)
if [ ! "$var" ]; then
    echo "Variable is empty"
fi

# Using [[ ]] (no quotes needed)
if [[ -n $var ]]; then
    echo "Not empty"
fi

# Default value (doesn't modify var)
echo "${var:-default}"

# Default value (modifies var)
echo "${var:=default}"
# Now $var is set to "default" if it was empty

# Use alternative value if set
echo "${var:+was_set}"

# Check if unset vs empty
if [ -z "${var+x}" ]; then
    echo "Variable is unset"
elif [ -z "$var" ]; then
    echo "Variable is set but empty"
else
    echo "Variable has value: $var"
fi

# Function with default argument
greet() {
    local name="${1:-World}"
    echo "Hello, $name"
}
```

## References
- GNU Bash Manual: Shell Parameter Expansion
