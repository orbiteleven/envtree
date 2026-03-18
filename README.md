# envtree

Sync .env files across git worktrees. Pull env files into branch worktrees, or push changes back.

## Install

```bash
npm install -g envtree-sync
```

Or with other package managers:

```bash
yarn global add envtree-sync
pnpm add -g envtree-sync
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

### Custom source directory

By default, envtree syncs from the main git worktree. If your .env files live elsewhere (e.g. a specific local checkout), set `source` in your config:

```json
{
  "source": "~/projects/myapp",
  "files": [
    ".env",
    "apps/*/.env.local"
  ]
}
```

This is useful when the main worktree doesn't have .env files (they're gitignored) and you want to pull from a directory where they already exist. Supports `~` for the home directory.

## Usage

From any worktree (not the main one):

```bash
envtree pull            # copy .env files into this worktree
envtree push            # copy .env files from this worktree to the source
```

You can also pass a directory directly, overriding both the config and main worktree:

```bash
envtree pull ~/myenvs   # pull from a specific directory
envtree push ~/myenvs   # push to a specific directory
```

Source priority: CLI argument > `source` in `.envtree.json` > main git worktree.

Push warns before overwriting files that differ in the target.

## Requirements

- bash, git
- A git repo with worktrees (`git worktree add`)
