---
type: concept
tags: [bash, strings, comparison]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I compare strings in bash?

## Overview
Bash provides multiple ways to compare strings using test operators within `[ ]` or the preferred `[[ ]]` construct.

## Key Points
- Use `[[ ]]` over `[ ]` for string comparisons (safer, no word splitting)
- `==` or `=` for equality (both work in `[[ ]]`)
- `!=` for inequality
- `-z` tests if string is empty (zero length)
- `-n` tests if string is not empty
- Always quote variables in `[ ]`, optional in `[[ ]]`
- `<` and `>` for lexicographic comparison (escape in `[ ]`)

## When to Use
- Validating user input
- Comparing configuration values
- Checking string equality in conditionals

## When NOT to Use
- For numeric comparison, use `(( ))` or `-eq`, `-lt`, `-gt`
- Single `[` requires quoting and escaping: `[ "$a" \< "$b" ]`

## Related Concepts
- [[How do I check if a variable is empty?]]
- [[Bash scripting MOC]]

## Examples

```bash
# Equality
if [[ "$str" == "hello" ]]; then
    echo "Match!"
fi

# Inequality
if [[ "$str" != "world" ]]; then
    echo "Not world"
fi

# Empty string
if [[ -z "$str" ]]; then
    echo "String is empty"
fi

# Non-empty string
if [[ -n "$str" ]]; then
    echo "String has content"
fi

# Pattern matching (glob)
if [[ "$str" == h* ]]; then
    echo "Starts with h"
fi

# Regex matching
if [[ "$str" =~ ^[0-9]+$ ]]; then
    echo "All digits"
fi

# Lexicographic comparison
if [[ "$a" < "$b" ]]; then
    echo "$a comes before $b"
fi

# Case insensitive comparison
shopt -s nocasematch
if [[ "$str" == "HELLO" ]]; then
    echo "Case-insensitive match"
fi
```

## References
- GNU Bash Manual: Conditional Constructs
