# Basic Workflow

## Essential Commands

### Check Status

```bash
# Show working tree status
git status

# Compact status
git status -s
git status --short

# Detailed status
git status -v
```

### Staging Changes

```bash
# Stage specific file
git add filename.py

# Stage all changes
git add .
git add -A
git add --all

# Stage by pattern
git add *.py
git add **/*.py

# Interactive staging
git add -i
git add --patch

# Update (stage modifications + deletions, not new files)
git add -u
git add --update
```

### Committing

```bash
# Commit staged changes
git commit -m "Commit message"

# Add and commit in one command (tracked files only)
git commit -am "Commit message"

# Amend last commit
git commit --amend
git commit --amend -m "New message"

# Empty commit (for CI triggers)
git commit --allow-empty -m "Trigger CI"
```

### Viewing Changes

```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --cached
git diff --staged

# Compare branches
git diff main..feature-branch
git diff main...feature-branch  # 3 dots = changes since branching

# Show file changes between commits
git diff abc123..def456 -- file.py

# Specific file
git diff -- file.py
```

## Typical Workflow

```bash
# 1. Check current state
git status

# 2. See what changed
git diff

# 3. Stage changes
git add -A

# 4. Review staged
git diff --cached

# 5. Commit
git commit -m "Add new feature"

# 6. Push to remote
git push origin main
```

## Commit Message Best Practices

```bash
# Good commit messages
git commit -m "Add user authentication with JWT tokens"
git commit -m "Fix: Resolve TypeError in data processing"
git commit -m "Update: Improve API response time by 30%"

# Conventional Commits format
git commit -m "feat: add password reset functionality"
git commit -m "fix: resolve NoneType error in parser"
git commit -m "docs: update API documentation"
git commit -m "refactor: simplify database queries"
git commit -m "test: add unit tests for auth module"
```

## Viewing History

```bash
# Basic log
git log

# Compact format
git log --oneline

# Graph + compact
git log --oneline --graph --decorate

# Filter by file
git log -- filename.py

# Filter by author
git log --author="John"

# Filter by date
git log --since="2024-01-01"
git log --after="2 weeks ago"

# Show diff for each commit
git log -p

# Show files changed
git log --stat

# Limit count
git log -n 5
git log -5
```

## Remove Files

```bash
# Remove from git and filesystem
git rm filename.py

# Remove from git only (keep file)
git rm --cached filename.py

# Remove directory
git rm -r directory/
```

## Move/Rename Files

```bash
# Rename file
git mv oldname.py newname.py

# Move file
git mv file.py directory/
```
