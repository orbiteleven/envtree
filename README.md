# envtree

Sync .env files across git worktrees. The main worktree is the source of truth — pull env files into branch worktrees, or push changes back.

## Install

Copy `envtree` somewhere on your PATH:

```bash
cp envtree /usr/local/bin/envtree
chmod +x /usr/local/bin/envtree
```

## Setup

In your repo, run:

```bash
envtree init
```

This creates `.envtree.json` in the repo root with glob patterns for your env files. Commit it so all worktrees share the config.

Example `.envtree.json`:

```json
{
  "files": [
    "apps/api/.env.local",
    "apps/*/.env"
  ]
}
```

## Usage

From any worktree (not the main one):

```bash
envtree pull    # copy .env files from main worktree into this one
envtree push    # copy .env files from this worktree back to main
```

Push warns before overwriting files that differ in the main worktree.

## Requirements

- bash, git, python3 (for JSON parsing)
- A git repo with worktrees (`git worktree add`)
