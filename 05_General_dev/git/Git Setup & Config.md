# Git Setup & Config

## Initial Setup

```bash
# Set global identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor code  # VS Code
git config --global core.editor vim    # Vim
git config --global core.editor nano    # Nano
```

## Initialize Repository

```bash
# New repository
git init

# Clone existing
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git  # SSH

# Shallow clone (faster)
git clone --depth 1 https://github.com/user/repo.git
```

## SSH Key Setup

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key to clipboard
# Windows: clip < ~/.ssh/id_ed25519.pub
# Mac: pbcopy < ~/.ssh/id_ed25519.pub
# Linux: xclip -sel clip < ~/.ssh/id_ed25519.pub
```

## Python .gitignore

Create `.gitignore` in project root:

```gitignore
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# Virtual environments
venv/
ENV/
env/
.venv/

# Distribution / packaging
*.egg-info/
dist/
build/
*.egg
PIPFILE
Pipfile.lock

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/

# IDEs
.vscode/
.idea/
*.swp
*.swo

# Jupyter Notebook
.ipynb_checkpoints/

# Environments
.env
.venv
env/
venv/

# mypy
.mypy_cache/
.pytype/
```

## Useful Configurations

```bash
# Pretty log output
git config --global alias.lg "log --oneline --graph --decorate --all"

# Default push behavior
git config --global push.default current

# Pull with rebase instead of merge
git config --global pull.rebase true

# Color output
git config --global color.ui auto

# List all config
git config --list --show-origin
```

## Repository-Specific Config

```bash
# Project-level config (no --global)
git config user.name "Project Name"
git config user.email "project@email.com"

# Editor for single repo
git config core.editor "code --wait"
```

## Check Configuration

```bash
# Show all config
git config --list

# Show specific setting
git config user.email

# Show config file locations
git config --list --show-origin

# Edit global config
git config --global --edit
```
