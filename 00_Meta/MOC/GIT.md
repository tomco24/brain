---
type: moc
tags: [moc, git, version-control]
created: 2026-02-21
updated: 2026-02-21
---

# Git MOC

## Overview
Git is a distributed version control system essential for Python development. It enables tracking changes, collaborating on code, managing branches, and maintaining project history. This MOC covers practical Git features specifically useful for Python developers.

**Prerequisites**: Command line basics, Python project structure

## Core Concepts

### Repository Fundamentals
- [[Git Setup & Config]] - Initial configuration, SSH keys, .gitignore for Python
- [[Basic Workflow]] - The fundamental add-commit-push cycle
- [[Remote Operations]] - Working with remotes, fetching, pushing

### Branching & Collaboration
- [[Branching & Merging]] - Creating branches, merging, rebasing
- [[GitHub Integration]] - PR workflow, forks, GitHub CLI
- [[Stashing]] - Temporarily saving changes

### Inspection & Recovery
- [[History & Inspection]] - log, diff, blame, bisect
- [[Undoing Changes]] - reset, revert, checkout specific changes
- [[Git Hooks for Python]] - pre-commit, pytest hooks, CI integration

## Implementation Patterns

### Daily Development Workflow
```bash
# Start of day
git pull origin main
git checkout -b feature/new-feature

# Throughout the day
git add -A
git commit -m "Description"
git push -u origin feature/new-feature

# Code review ready → create PR on GitHub
```

### Python-Specific Patterns
- `.gitignore` for Python (venv/, __pycache__/, *.pyc, .pytest_cache/)
- Pre-commit hooks for linting (ruff, black) and testing
- GitHub Actions for CI/CD pipelines

### Common Workflows
- **Feature Branch**: Create branch → develop → PR → merge
- **Hotfix**: Branch from main → fix → merge to main and develop
- **Release**: Branch from develop → tag → merge to main

## Advanced Topics

- [[Git Bisect]] - Binary search for bugs
- [[Submodules]] - Managing nested repositories
- [[Git Worktrees]] - Working on multiple branches simultaneously
- [[Git Attributes]] - Large file handling, line ending normalization

## Common Problems & Solutions

- **How do I undo a commit?**
  - `git reset --soft HEAD~1` (keep changes staged)
  - `git reset --hard HEAD~1` (discard changes - careful!)
- **How do I resolve merge conflicts?**
  - Edit conflicted files, then `git add` and `git commit`
  - Use `git mergetool` for visual resolution
- **How do I remove a file from git but keep it locally?**
  - `git rm --cached filename` (removes from staging, keeps file)
- **How do I see what changed in a commit?**
  - `git show <commit-hash>`
  - `git diff <commit-hash>^..<commit-hash>`
- **How do I clean up merged branches?**
  - `git branch -d branch-name` (local)
  - `git push origin --delete branch-name` (remote)

## Examples

- Python project `.gitignore` from real projects
- Pre-commit configuration with ruff, black, mypy
- GitHub Actions workflow for Python testing
- Commit message conventions (Conventional Commits)

## Related Areas

- [[Async programming]] - Using git with async Python projects
- [[Testing]] - Integrating with pytest
- [[Code Quality]] - Linting and formatting tools

## Learning Path

1. Start with [[Git Setup & Config]] and [[Basic Workflow]]
2. Practice with [[Branching & Merging]] on a test project
3. Learn [[History & Inspection]] to understand project evolution
4. Master [[Undoing Changes]] for recovery scenarios
5. Explore [[Git Hooks for Python]] to automate quality checks
6. Advance with [[GitHub Integration]] for collaboration

## References

- [Pro Git Book](https://git-scm.com/book/en/v2) - Free online book
- [GitHub Docs](https://docs.github.com/) - GitHub-specific guide
- [Atlassian Git Tutorial](https://www.atlassian.com/git) - Visual explanations
- [Conventional Commits](https://www.conventionalcommits.org/) - Commit message standard
