# Remote Operations

## Remote Basics

```bash
# List remotes
git remote -v

# Add remote
git remote add origin git@github.com:user/repo.git

# Rename remote
git remote rename origin upstream

# Remove remote
git remote remove origin
git remote rm origin

# Show remote details
git remote show origin

# Set remote URL
git remote set-url origin git@github.com:user/new-repo.git
```

## Fetching

```bash
# Fetch from default remote
git fetch

# Fetch from specific remote
git fetch origin

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch --prune
git fetch -p

# Shallow fetch (limit history)
git fetch --depth=1
git fetch --shallow-since="2024-01-01"
```

## Pulling

```bash
# Pull and merge
git pull

# Pull with rebase
git pull --rebase
git pull -r

# Pull specific branch
git pull origin feature-branch

# Force pull (overwrite local)
git fetch origin
git reset --hard origin/main
```

## Pushing

```bash
# Push to default remote
git push

# Push to specific remote
git push origin main

# Push and set upstream
git push -u origin feature-branch
git push --set-upstream origin feature-branch

# Push all branches
git push --all origin

# Push tags
git push origin v1.0.0
git push --tags

# Delete remote branch
git push origin --delete branch-name

# Force push (use with caution)
git push --force
git push --force-with-lease  # Safer
```

## Tracking Branches

```bash
# Set upstream for current branch
git push -u origin feature

# View tracking branches
git branch -vv

# Change tracked branch
git branch -u origin/branch
git branch --set-upstream-to=origin/branch

# Remove upstream tracking
git branch --unset-upstream
```

## Advanced Remote Operations

```bash
# Fetch from multiple remotes
git remote update

# Check if upstream is ahead/behind
git rev-list --left-right --count HEAD...origin/main

# See what's in remote but not local
git log HEAD..origin/main --oneline

# See what's in local but not remote
git log origin/main..HEAD --oneline

# Prune stale remote tracking branches
git remote prune origin

# Mirror repository
git push --mirror backup-repo
```

## Common Workflows

```bash
# Standard workflow
git clone git@github.com:user/repo.git
git checkout -b feature/my-feature
# ... work ...
git push -u origin feature/my-feature

# Sync with main
git fetch origin
git merge origin/main
# or
git pull origin main

# Keep feature branch updated
git fetch origin
git rebase origin/main
git push --force-with-lease
```

## Git Remote Internals

```bash
# See remote refs
git ls-remote origin
git ls-remote --heads origin

# See all refs including tags
git ls-remote
```

## Troubleshooting

```bash
# Reject push if upstream has changes
git config --global push.followTags true

# Automatic prune on fetch
git config --global fetch.prune true

# Handle push conflicts
git pull --rebase
# resolve conflicts, then
git push
```
