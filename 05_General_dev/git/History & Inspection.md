# History & Inspection

## Viewing Commits

```bash
# Basic log
git log

# Compact oneline
git log --oneline

# With graph
git log --graph --oneline --all

# With decorate (branch names)
git log --oneline --decorate

# Pretty formats
git log --pretty=format:"%h - %an, %ar : %s"
git log --pretty=fuller

# Date formats
git log --date=short
git log --date=relative
git log --date=iso
```

## Viewing Specific Commits

```bash
# Show commit details
git show abc123
git show HEAD
git show HEAD~3

# Show specific file at commit
git show abc123:filename.py

# Show last commit
git show HEAD
```

## File History

```bash
# Log for specific file
git log filename.py
git log -- filename.py

# Log with diffs
git log -p filename.py

# Show who changed each line
git blame filename.py

# Blame with email
git blame -e filename.py

# Blame specific lines
git blame -L 10,20 filename.py

# Ignore whitespace changes
git blame -w filename.py
```

## Diff Commands

```bash
# Working directory vs staging
git diff

# Staging vs last commit
git diff --cached
git diff --staged

# Working directory vs last commit
git diff HEAD

# Between commits
git diff abc123..def456
git diff main..feature

# Since branching point
git diff main...feature

# Specific file
git diff -- filename.py

# Stat summary
git diff --stat

# Word diff
git diff --word-diff

# Binary diffs
git diff --binary
```

## Searching History

```bash
# Search commit messages
git log --grep="fix"
git log --grep="feature" --grep="add" --all-match

# Search code changes
git log -S "function_name"
git log -G "regex_pattern"

# Search by author
git log --author="John"
git log --author="john@"

# Search by date
git log --since="2024-01-01"
git log --after="2 weeks ago"
git log --before="2024-12-31"
```

## Git Bisect (Binary Search)

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good v1.0.0

# Git checks out a commit - test it
# Mark as good or bad
git bisect good
git bisect bad

# When found, end bisect
git bisect reset
```

```bash
# Automated bisect
git bisect start HEAD v1.0.0
git bisect run python test.py
```

## Show Differences Between

```bash
# Branches
git diff main..feature

# Commits
git diff abc123..def456

# Stash
git diff stash@{0}
git diff stash@{0}^..stash@{0}

# Remote
git diff HEAD..origin/main
```

## Reflog (Recovery)

```bash
# View reflog (HEAD changes)
git reflog
git reflog show HEAD

# View for specific branch
git reflog show main

# Recover lost commit
git checkout abc123
git branch recovery-branch
```

## Shortcuts

```bash
# Show HEAD
git show HEAD

# Show parent (previous commit)
git show HEAD^

# Show grandparent
git show HEAD~3

# Show specific parent (merge commits)
git show HEAD^1  # First parent
git show HEAD^2  # Second parent
```
