# envtree

Sync .env files across git worktrees. Pull env files into branch worktrees, or push changes back.

## Install

Add it as a dev dependency in your project:

```bash
npm install --save-dev envtree-sync
```

Or with other package managers:

```bash
yarn add --dev envtree-sync
pnpm add --save-dev envtree-sync
```

Then run it with `npx envtree`, or add scripts to your `package.json`:

```json
{
  "scripts": {
    "env:pull": "envtree pull",
    "env:push": "envtree push"
  }
}
```

## Setup

In your repo, run:

```bash
npx envtree init
```

This creates `.envtree.json` in the repo root with a `source` directory and glob patterns for your env files. Commit it so all worktrees share the config.

Example `.envtree.json`:

```json
{
  "source": "~/projects/myapp",
  "files": [
    ".env",
    "apps/*/.env.local"
  ]
}
```

`source` is the local directory where your .env files live (e.g. a specific checkout). Supports `~` for the home directory.

## Usage

From any worktree:

```bash
npx envtree pull            # copy .env files into this worktree
npx envtree push            # copy .env files from this worktree to the source
```

You can also pass a directory directly, overriding the config source:

```bash
npx envtree pull ~/myenvs   # pull from a specific directory
npx envtree push ~/myenvs   # push to a specific directory
```

Source priority: CLI argument > `source` in `.envtree.json`.

Push warns before overwriting files that differ in the target.

## Requirements

- bash, git
- A git repo with worktrees (`git worktree add`)
