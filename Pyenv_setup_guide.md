# Pyenv Setup Guide — Python Version Management on Linux

**Document Type:** Technical Setup Guide  
**Platform:** Linux (Ubuntu/Debian-based)  
**Skill Level:** Beginner to Intermediate  
**Last Updated:** 2026

---

## What Is Pyenv and Why Use It?

When working on multiple Python projects, each project may require a different Python version. Installing Python directly on your system creates conflicts — upgrading Python for one project can break another.

**Pyenv solves this by letting you:**

- Install and switch between multiple Python versions without touching your system Python
- Set a specific Python version per project
- Keep your system clean and conflict-free

> **Key distinction:** Pyenv manages Python *versions*. It works alongside `venv` (which manages environments) and `pip` (which manages packages). They are not replacements for each other — they work together.

---

## Overview

| Tool | Purpose |
|---|---|
| `pyenv` | Installs and switches between Python versions |
| `venv` | Creates isolated environments per project |
| `pip` | Installs Python packages inside a venv |

---

## Phase 1 — System Dependencies

Before installing pyenv, install the required system libraries. These allow Python versions to build correctly from source.

```bash
sudo apt update
sudo apt install -y make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev \
libxmlsec1-dev libffi-dev liblzma-dev
```

---

## Phase 2 — Install Pyenv

This guide installs pyenv to a dedicated directory on an external or secondary drive. Adjust the path to match your system.

```bash
mkdir -p /media/YOUR_DRIVE/Programming/python
cd /media/YOUR_DRIVE/Programming/python
git clone https://github.com/pyenv/pyenv.git pyenv
```

> Replace `/media/YOUR_DRIVE/Programming/python` with your actual mount path throughout this guide.

---

## Phase 3 — Configure Your Shell Environment

Your terminal needs to know where pyenv is installed. This is done by editing your shell configuration file.

**Open your `.bashrc` file:**

```bash
nano ~/.bashrc
```

**Add these lines at the very bottom — no spaces before `export`:**

```bash
export PYENV_ROOT="/media/YOUR_DRIVE/Programming/python/pyenv"
export PATH="$PYENV_ROOT/bin:$PYENV_ROOT/shims:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

**Save and exit:**

```
CTRL + X → Y → Enter
```

**Reload your shell to apply changes:**

```bash
source ~/.bashrc
```

---

## Phase 4 — Install a Python Version

**Verify pyenv is working:**

```bash
pyenv --version
```

**Return to your home directory:**

```bash
cd ~
```

**Install your target Python version:**

```bash
pyenv install 3.12.2
```

> Python versions install to:  
> `/media/YOUR_DRIVE/Programming/python/pyenv/versions/`

**Useful pyenv commands:**

```bash
pyenv version              # Show currently active Python version
pyenv versions             # List all installed versions
pyenv install --list       # List all versions available to install
pyenv uninstall 3.11.7     # Remove an installed version
```

---

## Phase 5 — Project Workflow

Each project gets its own isolated Python environment. Follow this workflow for every new project.

**Navigate to your project directory:**

```bash
cd /media/YOUR_DRIVE/Programming/projects/YOUR_PROJECT_NAME
```

**Set the Python version for this project:**

```bash
pyenv local 3.12.2
```

**Create a virtual environment:**

```bash
python -m venv venv
```

**Activate the virtual environment:**

```bash
source venv/bin/activate
```

> Your terminal prompt will change to indicate the venv is active.

**Install project dependencies:**

```bash
pip install PACKAGE_NAME
```

---

## Phase 6 — Dependency Management

Managing dependencies ensures your project can be recreated on any machine.

**Save all installed packages to a requirements file:**

```bash
pip freeze > requirements.txt
```

**Install all dependencies from a requirements file:**

```bash
pip install -r requirements.txt
```

> The `-r` flag tells pip to read from the file and install everything listed.

---

## Recommended Directory Structure

```
/media/YOUR_DRIVE/Programming/
├── python/
│   └── pyenv/
├── projects/
│   ├── project-one/
│   ├── project-two/
│   └── project-three/
├── tools/
└── datasets/
```

Keeping pyenv and projects separated makes the setup portable and easier to maintain.

---

## Relocating Pyenv to a New Drive

If you need to move your pyenv installation to a different drive or machine:

**Create an archive of your pyenv installation:**

```bash
tar -czf pyenv.tar.gz -C /media/YOUR_DRIVE/Programming/python pyenv
```

**Extract to the new location:**

```bash
tar -xzf pyenv.tar.gz -C /media/NEW_DRIVE/python
```

**Point your terminal to the new location (temporary — current session only):**

```bash
export PYENV_ROOT="/media/NEW_DRIVE/python/pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

> To make this permanent, update the paths in your `~/.bashrc` file.

**Verify everything works:**

```bash
pyenv versions
python --version
python -m pip --version
```

> **Important:** Existing project virtual environments (`venv/`) will likely break after relocation because they store absolute paths to the Python interpreter. Recreate them after moving:
>
> ```bash
> python -m venv venv
> pip install -r requirements.txt
> ```

---

## Customizing the Virtual Environment Prompt

By default, activating a venv wraps your prompt in parentheses like `(venv)`. You can modify this appearance by editing the venv activation template.

**Open the activation script:**

```bash
nano /media/YOUR_DRIVE/Programming/python/pyenv/versions/3.12.10/lib/python3.12/venv/scripts/common/activate
```

**Find this block:**

```bash
if [ -z "${VIRTUAL_ENV_DISABLE_PROMPT:-}" ] ; then
_OLD_VIRTUAL_PS1="${PS1:-}"
PS1="("'**VENV_PROMPT**'") ${PS1:-}"
export PS1
fi
```

**Replace it with:**

```bash
if [ -z "${VIRTUAL_ENV_DISABLE_PROMPT:-}" ] ; then
_OLD_VIRTUAL_PS1="${PS1:-}"
PS1=**VENV_PROMPT**"${PS1:-}"
export PS1
fi
```

**Recreate the venv to apply the change:**

```bash
deactivate
rm -rf venv
python -m venv venv
source venv/bin/activate
```

---

## Common Mistakes

| Mistake | What Actually Happens |
|---|---|
| Thinking pyenv replaces venv | Pyenv manages versions. Venv manages environments. Both are needed. |
| Forgetting `pyenv local` in a project | Your project uses the wrong Python version silently. |
| Creating a venv before setting the Python version | Venv builds against the wrong interpreter. |
| Not restarting the shell after installing pyenv | Pyenv commands are not found. |
| System Python still running instead of pyenv | Shell config wasn't saved or sourced correctly. |

---

## When to Use Pyenv

**Use pyenv when:**
- Working on multiple projects that require different Python versions
- Contributing to external repositories with their own version requirements
- You need consistent Python versions across multiple machines

**Skip pyenv when:**
- You are writing a single quick script
- You only ever use one Python version and don't switch between projects

---

## Quick Reference

```bash
# Install a Python version
pyenv install 3.12.2

# Set version for current project
pyenv local 3.12.2

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate

# Save dependencies
pip freeze > requirements.txt

# Install from saved dependencies
pip install -r requirements.txt
```

---
