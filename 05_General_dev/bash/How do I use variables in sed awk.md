---
type: concept
tags: [bash, sed, awk, variables]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I use variables in sed awk?

## Overview
Passing shell variables to sed and awk requires special handling because these tools have their own variable systems and quoting rules.

## Key Points
- For sed: use double quotes and escape properly, or break out of quotes
- For awk: use `-v var=value` to pass variables (preferred)
- Alternative for awk: `-v var="$shell_var"` for shell variable assignment
- For sed with `/`: use different delimiter or escape: `s|$pattern|$replace|g`

## When to Use
- Dynamic search patterns from user input or variables
- Configurable replacement strings
- Processing files based on runtime values

## When NOT to Use
- Unquoted variables in sed/awk can cause syntax errors
- Backslash characters need extra escaping

## Related Concepts
- [[Bash scripting MOC]]
- [[AWK]]
- [[Sed]]

## Examples

### sed Examples

```bash
# Using double quotes with variable
name="world"
sed "s/hello/$name/" file.txt

# Break out of single quotes
name="world"
sed 's/hello/'"$name"'/' file.txt

# Using different delimiter for paths
path="/home/user"
sed "s|$path|/new/path|g" file.txt

# Escape special characters
replace="a&b"  # & is special in sed replacement
sed "s/pattern/${replace//&/\\&}/g" file.txt
```

### awk Examples

```bash
# Using -v flag (preferred)
var="hello"
awk -v val="$var" '{print val, $0}' file.txt

# Multiple variables
awk -v a="$var1" -v b="$var2" '{print a, b}' file.txt

# BEGIN block with -v
awk -v OFS="," '{print $1, $2}' file.txt

# Access environment variable directly (awk)
awk '{print ENVIRON["HOME"]}' file.txt

# Alternative: pass via environment
export MYVAR="value"
awk '{print ENVIRON["MYVAR"]}' file.txt

# Break out of quotes (less preferred)
awk '{print "'"$var"'"}' file.txt
```

## References
- GNU sed Manual
- GNU awk Manual
