# GitHub Integration

## GitHub CLI

### Installation & Setup

```bash
# Install on Windows
winget install GitHub.cli

# Login
gh auth login

# Check status
gh auth status
```

### Repository Operations

```bash
# Create repository
gh repo create my-repo --public
gh repo create my-repo --private

# Clone repository
gh repo clone user/repo

# Fork repository
gh repo fork

# View repository info
gh repo view
gh repo view user/repo
```

## Pull Requests

### Create PR

```bash
# Create PR from command line
gh pr create
gh pr create --title "Add feature"
gh pr create --body "Description"

# Create PR from branch
gh pr create --fill  # Auto-fill title and body

# Create draft PR
gh pr create --draft
```

### Manage PRs

```bash
# List PRs
gh pr list

# View PR
gh pr view 123
gh pr view --web  # Open in browser

# Check PR status
gh pr status

# Checkout PR branch locally
gh pr checkout 123

# Merge PR
gh pr merge
gh pr merge --admin --squash
gh pr merge --merge  # No-squash merge
```

### PR Reviews

```bash
# Approve PR
gh pr review --approve

# Request changes
gh pr review --request-changes --body "Needs fixes"

# Add comment
gh pr review --comment --body "Looks good"
```

## Issues

```bash
# List issues
gh issue list

# Create issue
gh issue create --title "Bug in login"
gh issue create --title "New feature" --body "Description"

# View issue
gh issue view 123
gh issue view 123 --web

# Close issue
gh issue close 123
```

## Releases

```bash
# List releases
gh release list

# Create release
gh release create v1.0.0
gh release create v1.0.0 --notes "Release notes"

# Upload assets
gh release upload v1.0.0 dist/package.tar.gz
```

## Gists

```bash
# Create gist
echo "code" | gh gist create

# List gists
gh gist list

# View gist
gh gist view abc123
```

## GitHub Actions

```bash
# List workflow runs
gh run list

# View workflow run
gh run view 12345
gh run view --log

# Rerun workflow
gh run rerun 12345

# Download artifacts
gh run download 12345
```

## Authentication

```bash
# Use SSH
gh auth login
# Select SSH

# Use token
gh auth login
# Paste token when asked

# Logout
gh auth logout
```

## Useful GitHub Workflows

### Python GitHub Actions Template

```yaml
name: Python CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest pytest-cov
          pip install -r requirements.txt
      
      - name: Run tests
        run: pytest --cov=. --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

### Pre-commit with GitHub Actions

```yaml
name: Lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - uses: pre-commit/action@v3.0.1
```

## Fork Workflow

```bash
# Fork on GitHub, then clone
git clone git@github.com:yourusername/repo.git
cd repo

# Add upstream
git remote add upstream git@github.com:original/repo.git

# Keep updated
git fetch upstream
git merge upstream/main

# Push to your fork
git push origin main

# Create PR
gh pr create --base original:main --head your-branch
```
