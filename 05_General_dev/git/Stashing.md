# Stashing

## Basic Stash

```bash
# Stash changes
git stash

# Stash with message
git stash save "Work in progress"
git stash push -m "Work in progress"

# Stash including untracked files
git stash -u
git stash --include-untracked

# Stash including ignored files
git stash -a
git stash --all
```

## Manage Stashes

```bash
# List stashes
git stash list

# Show stash contents
git stash show
git stash show -p  # full diff
git stash show stash@{0} -p

# Apply most recent stash
git stash pop

# Apply specific stash
git stash pop stash@{2}

# Apply without removing from stash list
git stash apply
git stash apply stash@{1}
```

## Stash Branch

```bash
# Create branch from stash
git stash branch new-branch
git stash branch new-branch stash@{0}
```

## Clean Stash

```bash
# Drop most recent stash
git stash drop

# Drop specific stash
git stash drop stash@{2}

# Clear all stashes
git stash clear
```

## Partial Stash

```bash
# Interactively stash specific files/changes
git stash -p
git stash --patch

# Stash specific file
git stash push path/to/file.py
git stash push -m "message" -- path/to/file.py
```

## Useful Stash Patterns

```bash
# Quick save current work
git stash

# Pull latest and reapply
git stash && git pull && git stash pop

# Or use pull with stash
git pull --rebase && git stash pop

# Save work before switching branches
git stash push -m "feature work in progress"
git checkout other-branch
git stash pop
```

## Stash Configuration

```bash
# Enable stash auto-save
git config --global stash.showPatch true
git config --global stash.showStat true
```

## Show Working Tree Status in Stash

```bash
# Show what a stash contains
git stash show

# Show detailed diff
git stash show -p

# Show specific stash
git stash show stash@{0} -p
```

## Common Workflows

```bash
# Interrupt current work for urgent fix
git stash
git checkout main
# ... do fix ...
git checkout feature-branch
git stash pop

# Keep stashed changes as backup
git stash push -m "backup before risky operation"
# ... do risky operation ...
# If failed:
git stash pop

# Apply only certain files from stash
git stash show -p | git apply - <pathspec>
```
