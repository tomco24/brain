---
type: concept
tags: [bash, read, file-processing]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I read a file line by line?

## Overview
The canonical way to read a file line by line in bash is using `while IFS= read -r` with input redirection. This properly handles spaces, backslashes, and special characters.

## Key Points
- Always use `IFS=` to prevent trimming leading/trailing whitespace
- Always use `-r` to prevent backslash interpretation
- Use `|| [ -n "$line" ]` to handle files without final newline
- Avoid `for line in $(cat file)` - it breaks on spaces and is slow

## When to Use
- Processing structured text files
- Parsing logs line by line
- Reading configuration files
- When you need to perform operations per line

## When NOT to Use
- For simple file operations, use `grep`, `sed`, `awk` directly
- For large files, consider `awk` or `sed` for better performance

## Related Concepts
- [[How do I handle spaces in filenames?]]
- [[Bash scripting MOC]]

## Examples

```bash
# Basic line-by-line reading (correct way)
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Handle files without trailing newline
while IFS= read -r line || [ -n "$line" ]; do
    echo "$line"
done < file.txt

# Read into multiple variables (CSV-like)
while IFS=, read -r col1 col2 col3; do
    echo "$col1 - $col2 - $col3"
done < data.csv

# Process substitution (from command output)
while IFS= read -r line; do
    echo "$line"
done < <(grep pattern file.txt)

# Pipe input
grep pattern file.txt | while IFS= read -r line; do
    echo "$line"
done

# Read specific number of characters
while IFS= read -r -n1 char; do
    echo "Char: $char"
done < file.txt

# Read with timeout
while IFS= read -r -t 5 line; do
    echo "$line"
done < file.txt

# Read from file descriptor (for multiple files)
exec 3< file.txt
while IFS= read -r -u 3 line; do
    echo "$line"
done
exec 3<&-
```

## References
- GNU Bash Manual: read builtin
- Bash FAQ: How can I read a file line-by-line?
