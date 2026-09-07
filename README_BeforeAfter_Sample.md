## README Transformation Sample

This is a representative before/after example showing the MainbyteLabs documentation standard applied to a typical weak open\-source README. Used as a portfolio sample for the $50 README Documentation service.

* * *

## BEFORE

```markdown
# quicknote

Quicknote is a small command line tool I built because I kept losing notes I'd jot down while working in the terminal and didn't want to open a separate app or deal with syncing to some cloud service every time. It's written in Python and uses SQLite under the hood to store everything locally, so there's no account, no server, nothing to configure really. I've been using it daily for about a year now and figured I'd clean it up and put it on GitHub in case anyone else finds it useful.

Right now it supports adding notes with tags, searching through them, and listing recent ones. I want to eventually add export and maybe a sync option but haven't gotten around to it.

## Requirements

- Python 3
- SQLite3 (usually already installed)

## Getting Started

Clone the repo and install the requirements:
git clone https://github.com/jsmith/quicknote.git
cd quicknote
pip install \-r requirements.txt
## Features

- Add notes
- Tag notes
- Search
- SQLite backend
- Fast

## Usage

Run `python main.py` and follow the prompts.

## Contributing

Feel free to open a PR.

## License

MIT

```

**What's actually wrong with this:\*\* it reads like a finished project — full sentences, a Requirements section, an install step. That's exactly why it underperforms: nothing here is scannable. A developer has to *read* the paragraph to learn what it does instead of catching it in three seconds. There's no command syntax anywhere, so "run main.py and follow the prompts" tells you nothing about what to actually type. The install step is incomplete (which Python version? what's in requirements.txt?). The feature list is five one\-word fragments with no explanation. And there's no visual of how the tool works — just prose. This is the profile of 80% of real underperforming READMEs: not empty, just badly organized and impossible to skim.

## AFTER {#after}

```markdown
# quicknote

Save and search notes from your terminal — no context switching, no GUI, no cloud account required.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Why quicknote

If you live in the terminal, quicknote lets you capture a thought, a command, or a snippet without opening another app. Notes are stored locally in SQLite, tagged for fast retrieval, and searchable in milliseconds.

## Requirements

- Python 3.9+
- SQLite3 (bundled with Python on most systems)

## Install

    git clone https://github.com/jsmith/quicknote.git
    cd quicknote
    pip install -r requirements.txt

## Usage

Save a note:

    python main.py add "remember to rotate the API key" --tag ops

Search notes:

    python main.py search ops

Output:

    [2026-09-03] remember to rotate the API key  #ops

## How it works

    flowchart LR
        A[add command] --> B[SQLite store]
        B --> C[Tag index]
        D[search command] --> C
        C --> E[Ranked results in terminal]

## Features

| Command | What it does |
|---|---|
| `add` | Save a new note with optional tags |
| `search` | Full-text search across all notes |
| `list` | Show recent notes |

Planned: `export` to dump all notes to a single markdown file.

## Contributing

Feel free to open a PR.

## License

MIT
```

**What changed:** a one\-line pitch that says what it does and why it matters, a Requirements section so there's no guessing about Python version, the exact install commands (unchanged from the original distribution method — no false claims about a package that was never published), copy\-pasteable usage commands with real output shown, a Mermaid diagram showing the actual data flow (add → store → index, search → index → results) instead of two boxes for decoration, and a feature table that separates what's shipped from what's planned. This is what turns a README from "technically has the info somewhere" into "I can evaluate and try this in under a minute."
