# Undoing Changes

## Unstage Files

```bash
# Unstage specific file
git reset HEAD filename.py

# Unstage all files
git reset HEAD
git reset
```

## Undo Working Directory Changes

```bash
# Discard changes in working directory
git checkout -- filename.py
git restore filename.py  # Git 2.23+

# Discard all changes
git checkout -- .
git restore .
```

## Undo Commits

### Soft Reset (keep changes staged)

```bash
# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Undo last 3 commits
git reset --soft HEAD~3
```

### Mixed Reset (keep changes unstaged)

```bash
# Undo last commit, keep changes in working directory
git reset --mixed HEAD~1
git reset HEAD~1  # default
```

### Hard Reset (discard all changes)

```bash
# Undo last commit, discard ALL changes
git reset --hard HEAD~1

# Undo last 3 commits
git reset --hard HEAD~3

# ⚠️ DANGER - discard all uncommitted changes
git reset --hard
```

## Revert (Create New Commit Undoing Changes)

```bash
# Revert last commit (creates new commit)
git revert HEAD

# Revert specific commit
git revert abc123

# Revert without auto-commit
git revert -n abc123
git revert --no-commit abc123

# Revert merge commit (specify parent)
git revert -m 1 abc123  # Keep first parent's changes
```

## Reset vs Revert

| Reset | Revert |
|-------|--------|
| Rewrites history | Creates new commit |
| Not safe for shared branches | Safe for shared branches |
| Moves branch pointer | Doesn't change history |
| Use locally | Use on shared branches |

## Recover Lost Commits

```bash
# Find commit in reflog
git reflog

# Create branch at lost commit
git branch recovery-branch abc123

# Or checkout and create branch
git checkout abc123
git checkout -b recovery-branch
```

## Checkout Files from Previous Commits

```bash
# Restore file from specific commit
git checkout abc123 -- filename.py

# Restore from parent of HEAD
git checkout HEAD~1 -- filename.py
git restore --source=HEAD~1 filename.py
```

## Undo Specific Changes in File

```bash
# Checkout only specific lines/changes
git checkout -p filename.py

# Or interactively restore
git restore -p filename.py
```

## Empty / Amend Commit

```bash
# Amend last commit message
git commit --amend -m "New message"

# Amend and add forgotten files
git add forgotten.py
git commit --amend --no-edit

# Create empty commit (CI trigger)
git commit --allow-empty -m "Trigger build"
```

## Git Restore (Git 2.23+)

```bash
# Discard unstaged changes
git restore filename.py

# Unstage file
git restore --staged filename.py

# Both unstage and discard
git restore -S filename.py
git restore -W filename.py

# Restore from specific commit
git restore --source=abc123 filename.py

# Restore to state 3 commits ago
git restore --source=HEAD~3 filename.py
```
