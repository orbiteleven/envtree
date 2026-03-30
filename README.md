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

### Other tools

For any tool that supports worktree setup scripts or hooks, add `npx envtree pull` as a post-creation step. If the tool doesn't have hooks, you can add it to a project-level setup script:

```bash
#!/bin/bash
# setup.sh — run after worktree creation
npx envtree pull
```

### Tips

- **Commit `.envtree.json`** so worktrees created by agents inherit the config automatically.
- **Set `source`** to your primary checkout (e.g. `~/projects/myapp`) so agents always pull from a known-good location.
- Use `--debug` to troubleshoot: `npx envtree pull --debug`

## Requirements

- bash, git
- A git repo with worktrees (`git worktree add`)
