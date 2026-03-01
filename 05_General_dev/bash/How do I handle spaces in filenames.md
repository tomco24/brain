---
type: concept
tags: [bash, filenames, quoting]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I handle spaces in filenames?

## Overview
Spaces in filenames cause word splitting in bash unless properly quoted. Always quote variables containing filenames to prevent unexpected behavior.

## Key Points
- Always wrap filename variables in double quotes: `"$file"`
- Use `find -print0` with `xargs -0` for null-delimited processing
- Set `IFS= read -r` when reading filenames with spaces
- Avoid backticks for command substitution with filenames

## When to Use
- Processing user-provided filenames
- Working with directories that may contain spaces
- Looping over files with `for file in *`

## When NOT to Use
- Single quotes prevent variable expansion - use double quotes instead
- Unquoted `$var` will split on spaces and cause errors

## Related Concepts
- [[How do I read a file line by line?]]
- [[Bash scripting MOC]]

## Examples

```bash
# Wrong - breaks on spaces
for file in $(ls); do
    cat $file
done

# Correct - handles spaces
for file in *; do
    cat "$file"
done

# Find with spaces
find . -type f -print0 | xargs -0 rm

# Reading filenames
find . -type f | while IFS= read -r file; do
    echo "Processing: $file"
done
```

## References
- GNU Bash Manual: Word Splitting
