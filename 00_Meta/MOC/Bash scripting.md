---
type: moc
tags: [moc, bash, scripting, shell]
created: 2026-02-21
updated: 2026-02-21
---

# Bash Scripting MOC

## Overview
Bash (Bourne Again Shell) is the default shell on most Linux systems and macOS. It combines powerful scripting capabilities with access to Unix core utilities for automation, text processing, and system administration.

**Prerequisites**: [[Command Line Basics]], [[File System Structure]]

## Core Concepts

### Variables and Data Types
- Variable declaration and assignment: `var="value"`
- Parameter expansion: `$var`, `${var}`
- Special variables: `$0`, `$1-$9`, `$#`, `$@`, `$?`, `$$`
- Arrays: `arr=(one two three)`, `${arr[@]}`, `${arr[0]}`

### Control Flow
- Conditionals: `if/elif/else`, `case` statements
- Loops: `for`, `while`, `until`, `select`
- Test expressions: `[ ]`, `[[ ]]`, `(( ))`

### Functions
- Definition: `function name() { ... }` or `name() { ... }`
- Parameters: `$1`, `$2`, `$@`
- Return values: `return` (exit status 0-255), `echo` for data

### Input/Output
- Redirection: `>`, `>>`, `<`, `2>`, `&>`
- Pipes: `|`
- Here documents: `<<EOF ... EOF`
- Here strings: `<<<`

### Quoting and Expansion
- Single quotes: literal interpretation
- Double quotes: variable expansion
- Escape sequences: `\`
- Globbing: `*`, `?`, `[ ]`
- Tilde expansion: `~`, `~/path`

## Common Binaries

### Text Processing
| Command | Description | Key Flags |
|---------|-------------|-----------|
| `grep` | Pattern matching | `-i`, `-r`, `-v`, `-n`, `-E`, `-o` |
| `sed` | Stream editor | `-i`, `s/old/new/g`, `-d` |
| `awk` | Text processing/awkward | `-F`, `print`, `$1`, `NR` |
| `sort` | Sort lines | `-n`, `-r`, `-k`, `-u` |
| `uniq` | Remove duplicates | `-c`, `-d`, `-i` |
| `head` | First lines | `-n`, `-c` |
| `tail` | Last lines | `-n`, `-f` (follow) |
| `cut` | Extract columns | `-d`, `-f`, `-c` |
| `tr` | Character translation | `-d`, `s///` |
| `wc` | Word/line count | `-l`, `-w`, `-c` |
| `cat` | Concatenate/display | `-n`, `-s` |

### File Operations
| Command | Description | Key Flags |
|---------|-------------|-----------|
| `ls` | List directory | `-l`, `-a`, `-R`, `-h`, `-1` |
| `find` | Find files | `-name`, `-type`, `-exec`, `-mtime` |
| `xargs` | Build arguments | `-I`, `-n`, `-0` |
| `cp` | Copy | `-r`, `-i`, `-v`, `-u` |
| `mv` | Move/rename | `-i`, `-v` |
| `rm` | Remove | `-r`, `-f`, `-i` |
| `mkdir` | Make directory | `-p`, `-v` |
| `chmod` | Change mode | `+x`, `755`, `644` |
| `chown` | Change owner | `-R` |

### System & Info
| Command | Description | Key Flags |
|---------|-------------|-----------|
| `echo` | Print text | `-e`, `-n` |
| `pwd` | Print working directory | `-P` |
| `cd` | Change directory | `-`, `~` |
| `which` | Find executable | `-a` |
| `type` | Command type | `-a` |
| `man` | Manual pages | |
| `date` | Date/time | `+%Y-%m-%d` |

### Combining Commands
- `tee`: Read from stdin, write to file and stdout
- `xargs`: Convert stdin to arguments
- `exec`: Replace shell with command

## Implementation Patterns

### Piping Patterns
```bash
# Filter and transform
cat file | grep pattern | sed 's/foo/bar/' | awk '{print $2}'

# Count unique occurrences
cat log.txt | grep ERROR | cut -d' ' -f5 | sort | uniq -c | sort -rn

# Process files found by find
find . -name "*.txt" -exec cat {} \; | grep pattern
```

### Conditional Patterns
```bash
# Check if file exists
if [ -f "$file" ]; then
    echo "File exists"
fi

# Check if command succeeded
if grep -q pattern file; then
    echo "Found"
fi

# Default value
${var:-default}
```

### Looping Patterns
```bash
# Over files
for file in *.txt; do
    echo "Processing $file"
done

# Over command output
for line in $(cat file); do
    echo "$line"
done

# While read
while IFS= read -r line; do
    echo "$line"
done < file
```

### Function Patterns
```bash
# Function with arguments and return
function greet() {
    local name="$1"
    echo "Hello, $name"
    return 0
}

# Function that returns data via stdout
get_date() {
    date +%Y-%m-%d
}
today=$(get_date)
```

## Advanced Topics

### Regular Expressions
- Basic regex (BRE) vs Extended regex (ERE)
- `grep -E` for ERE
- Common patterns: `^`, `$`, `.`, `*`, `+`, `?`, `[ ]`, `[^ ]`

### Process Substitution
- `<(command)` - use command output as file
- `>(command)` - send input to command
```bash
diff <(sort file1) <(sort file2)
```

### Subshells and Grouping
- Subshell: `(commands)` - runs in child process
- Grouping: `{ commands; }` - runs in current shell

### Error Handling
- `set -e` - exit on error
- `set -u` - exit on undefined variable
- `set -o pipefail` - catch pipe errors
- Trap: `trap 'cleanup' EXIT`

### Debugging
- `set -x` - trace execution
- `set -v` - print input lines
- `bash -x script.sh` - debug mode

## Common Problems & Solutions

- [[05_General_dev/bash/How do I handle spaces in filenames]]
- [[05_General_dev/bash/How do I redirect stderr and stdout separately]]
- [[05_General_dev/bash/How do I compare strings in bash]]
- [[05_General_dev/bash/How do I get the exit code of a command]]
- [[05_General_dev/bash/How do I use variables in sed awk]]
- [[05_General_dev/bash/How do I read a file line by line]]
- [[05_General_dev/bash/How do I check if a variable is empty]]
- [[05_General_dev/bash/How do I run a command in parallel]]

## Examples

- [[Backup Script]]
- [[Log Parser]]
- [[File Organizer]]
- [[System Health Check]]

## Related Areas

- [[Linux Command Line]]
- [[Cron Jobs]]
- [[AWK]]
- [[Sed]]
- [[Regex]]
- [[Git]] (for shell integration)

## Learning Path

1. Start with [[Core Concepts]] - variables, conditionals, loops
2. Then explore [[Common Binaries]] - grep, sed, awk, find
3. Practice with [[Implementation Patterns]] - piping, functions
4. Move to [[Advanced Topics]] - regex, debugging, error handling

## References

- GNU Bash Manual: https://www.gnu.org/software/bash/manual/
- Linux man pages: `man bash`, `man grep`, `man sed`, `man awk`
- [[Literature Note - Bash Cookbook]]
- [[Literature Note - Linux Command Line]]
