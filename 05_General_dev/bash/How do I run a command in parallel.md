---
type: concept
tags: [bash, parallel, concurrency]
created: 2026-02-21
modified: 2026-02-21
status: draft
---

# How do I run a command in parallel?

## Overview
Bash supports running commands in parallel using background jobs (`&`), `xargs -P`, and tools like GNU Parallel for more complex parallelization.

## Key Points
- `&` runs a command in background, `wait` blocks until all complete
- `$!` captures PID of last background job
- `xargs -P N` runs N processes in parallel
- GNU Parallel provides advanced features: job control, progress, retries
- Limit parallel jobs to available CPU cores to avoid overload

## When to Use
- Processing multiple independent files simultaneously
- Running slow operations (API calls, downloads) in parallel
- Speeding up CPU-bound tasks on multi-core systems

## When NOT to Use
- When tasks depend on each other's output
- When order of execution matters
- When shared resources could cause race conditions

## Related Concepts
- [[Bash scripting MOC]]
- [[Process Management]]

## Examples

### Background Jobs with &

```bash
# Run in background
sleep 5 &
echo "Background PID: $!"

# Run multiple and wait
sleep 3 &
sleep 5 &
wait
echo "All done"

# Wait for specific job
sleep 5 &
pid=$!
# ... do other work ...
wait $pid
echo "Specific job done"

# Collect exit codes
sleep 3 &
pids+=($!)
sleep 5 &
pids+=($!)

for pid in "${pids[@]}"; do
    wait $pid
    echo "PID $pid exited with $?"
done
```

### xargs -P

```bash
# Process files in parallel (4 at a time)
find . -name "*.txt" | xargs -P 4 -I {} process_file {}

# With line-by-line input
cat urls.txt | xargs -P 8 -I {} curl -s {} > /dev/null

# Combined with find
find . -type f -print0 | xargs -0 -P 4 -n 1 gzip
```

### GNU Parallel

```bash
# Basic parallel
parallel echo {} ::: a b c d

# Process files
parallel gzip {} ::: *.txt

# From stdin
cat files.txt | parallel process {}

# Limit jobs
parallel -j 4 process {} ::: *.txt

# Preserve order
parallel -k echo {} ::: 3 2 1

# Progress bar
parallel --bar process {} ::: *.txt

# Retry on failure
parallel --retries 3 curl {} ::: http://example.com

# Distribute across remote servers
parallel -S server1,server2 process {} ::: *.txt
```

### Job Control Functions

```bash
# Parallel with max jobs
max_jobs=4
job_count=0

for file in *.txt; do
    process "$file" &
    ((job_count++))
    
    if ((job_count >= max_jobs)); then
        wait -n  # Wait for any one job
        ((job_count--))
    fi
done
wait
```

## References
- GNU Parallel Documentation: https://www.gnu.org/software/parallel/
- GNU Bash Manual: Job Control
