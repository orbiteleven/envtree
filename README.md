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

This creates `.envtree.json` in the repo root with the glob patterns for your env files. Commit it so all worktrees share the config.

Example `.envtree.json`:

```json
{
  "files": [
    ".env",
    "apps/*/.env.local"
  ]
}
```

## Source directory

The source is the local directory your .env files are copied from and pushed back to — usually your primary checkout. Since `.envtree.json` is committed, the recommended way to set it is the `ENVTREE_SOURCE` environment variable, so each machine points at its own path.

### bash / zsh

Add to `~/.bashrc` or `~/.zshrc`:

```bash
export ENVTREE_SOURCE="$HOME/projects/myapp"
```

### fish

`set -Ux` stores it as a universal variable, so it's set once and persists across sessions:

```fish
set -Ux ENVTREE_SOURCE "$HOME/projects/myapp"
```

### Windows (PowerShell)

envtree needs bash, so run it under Git Bash or WSL and set the variable there. To set it for PowerShell-launched processes instead:

```powershell
[Environment]::SetEnvironmentVariable("ENVTREE_SOURCE", "$HOME\projects\myapp", "User")
```

### Per-project, with direnv

If you work in several repos with different sources, put it in a `.envrc` (add `.envrc` to `.gitignore`):

```bash
export ENVTREE_SOURCE="$HOME/projects/myapp"
```

### Falling back to the config file

You can still set `source` in `.envtree.json` — useful for a solo project, or when every machine really does share a layout. It accepts `~` and environment variable references, so a committed config can stay machine-independent:

```json
{
  "source": "$PROJECTS/myapp",
  "files": [".env"]
}
```

Referencing a variable that isn't set is an error rather than a silently wrong path.

**Source priority:** CLI argument > `$ENVTREE_SOURCE` > `source` in `.envtree.json`.

## Usage

From any worktree:

```bash
npx envtree pull            # copy .env files into this worktree
npx envtree push            # copy .env files from this worktree to the source
```

You can also pass a directory directly, overriding the configured source:

```bash
npx envtree pull ~/myenvs   # pull from a specific directory
npx envtree push ~/myenvs   # push to a specific directory
```

Push warns before overwriting files that differ in the target.

## Using with LLM coding agents

LLM-based coding tools like [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Codex](https://github.com/openai/codex), and others often run sub-agents in isolated git worktrees. These worktrees won't have your `.env` files, which can break builds, tests, and dev servers the agent tries to run.

envtree fixes this. Add `npx envtree pull` to your agent's setup so env files are available before anything else runs.

### Claude Code

In your `CLAUDE.md` or `.claude/settings.json`, add a hook that runs on worktree creation:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "EnterWorktree",
        "hooks": [
          {
            "type": "command",
            "command": "npx envtree pull"
          }
        ]
      }
    ]
  }
}
```

This runs `envtree pull` automatically whenever Claude Code creates a worktree for a sub-agent.

Hooks run non-interactively, so they don't load `~/.zshrc` or `~/.bashrc` — a shell-profile `ENVTREE_SOURCE` may not reach them. Set it in the same `.claude/settings.json` to be sure:

```json
{
  "env": {
    "ENVTREE_SOURCE": "/Users/you/projects/myapp"
  }
}
```

Use `.claude/settings.local.json` (gitignored) for this if the repo is shared, so your path stays off the commit.

### Other tools

For any tool that supports worktree setup scripts or hooks, add `npx envtree pull` as a post-creation step. If the tool doesn't have hooks, you can add it to a project-level setup script:

```bash
#!/bin/bash
# setup.sh — run after worktree creation
npx envtree pull
```

### Tips

- **Commit `.envtree.json`** so worktrees created by agents inherit the config automatically.
- **Point `ENVTREE_SOURCE` at your primary checkout** so agents always pull from a known-good location. Make sure it's exported where the agent runs, not just in your interactive shell.
- Use `--debug` to troubleshoot: `npx envtree pull --debug`. It prints which source was used and where it came from.

## Requirements

- bash, git
- A git repo with worktrees (`git worktree add`)
