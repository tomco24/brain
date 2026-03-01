# Branching & Merging

## Branch Basics

```bash
# List local branches
git branch

# List remote branches
git branch -r

# List all branches
git branch -a

# Create new branch (stay on current)
git branch feature-name

# Create and switch to new branch
git checkout -b feature-name
git switch -c feature-name  # Git 2.23+

# Delete branch (merged)
git branch -d branch-name

# Delete branch (unmerged - force)
git branch -D branch-name
```

## Switching Branches

```bash
# Switch branch
git checkout branch-name
git switch branch-name  # Git 2.23+

# Switch to previous branch
git checkout -
git switch -

# Detached HEAD (look at specific commit)
git checkout abc123
git checkout v1.0.0
```

## Merging

```bash
# Merge branch into current
git merge feature-branch

# Merge with no fast-forward (always creates merge commit)
git merge --no-ff feature-branch

# Abort merge if conflicts
git merge --abort

# Continue after resolving conflicts
git merge --continue
```

## Rebasing

```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase (last 3 commits)
git rebase -i HEAD~3

# Continue rebase after resolving
git rebase --continue

# Skip problematic commit
git rebase --skip

# Abort rebase
git rebase --abort
```

## Rebase vs Merge

| Rebase | Merge |
|--------|-------|
| Linear history | Preserves branch structure |
| Rewrites commits | Creates merge commit |
| Use locally | Use for shared branches |
| Never rebase public branches | Safe for any branch |

```bash
# Safe workflow: rebase before pushing
git checkout feature
git fetch origin
git rebase origin/main
git push --force-with-lease
```

## Handling Merge Conflicts

```bash
# See conflicting files
git status

# View conflicts
git diff

# Accept our version
git checkout --ours filename.py

# Accept their version
git checkout --theirs filename.py

# Accept both (as merge conflict markers)
git checkout --merge filename.py

# After resolving
git add filename.py
git commit
```

## Using Merge Tools

```bash
# Configure merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Launch merge tool
git mergetool

# Keep .orig backup files
git config --global mergetool.keepBackup false
```

## Delete Remote Branch

```bash
# Delete remote branch
git push origin --delete branch-name

# Delete local remote-tracking branch
git branch -dr origin/branch-name
```

## Rename Branch

```bash
# Rename current branch
git branch -m new-name

# Rename specific branch
git branch -m old-name new-name
```

## Track Remote Branch

```bash
# Set upstream for current branch
git push -u origin feature-branch
git push --set-upstream origin feature-branch

# Show tracking relationship
git branch -vv

# Change remote branch tracked
git branch -u origin/new-branch
```

## Cherry-Pick

```bash
# Pick single commit
git cherry-pick abc123

# Pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick without committing
git cherry-pick -n abc123
git cherry-pick --no-commit abc123
```
