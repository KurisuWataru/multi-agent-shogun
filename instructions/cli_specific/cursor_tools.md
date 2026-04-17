# Cursor CLI Tools

This section describes Cursor CLI-specific tools and behavior for the tmux-based Shogun system.

## Overview

Cursor CLI (the `agent` executable) runs the same agent engine as the editor inside a terminal session.

- **Launch**: `agent`
- **Auth**: `agent login`
- **Status**: `agent status`
- **Default interactive mode**: Agent mode
- **Other modes**: Plan mode, Ask mode

## Tool Usage

Cursor CLI exposes file editing, search, shell execution, web access, and MCP integrations in the terminal.

- **File operations**: Read and edit files directly in the workspace
- **Search**: Codebase-aware search and context selection with `@`
- **Shell commands**: Command execution with approval controls
- **Web access**: Web search and fetch support when enabled
- **MCP**: Reuses the same MCP configuration as the editor via `mcp.json`

## Rules Loading

Cursor CLI automatically applies the same instruction layers as the editor:

1. `.cursor/rules/*.mdc`
2. `AGENTS.md`
3. `CLAUDE.md`

For `multi-agent-shogun`, the repository-wide rules are shared, and role-specific instructions are loaded separately from `instructions/generated/cursor-*.md`.

## Modes

Switch modes with slash commands or flags:

| Mode | Slash command | Launch flag |
|------|---------------|-------------|
| Agent | default | default |
| Plan | `/plan` | `--plan` / `--mode=plan` |
| Ask | `/ask` | `--mode=ask` |

The Shogun system should launch Cursor workers in **Agent** mode so they can edit files and run commands autonomously.

## Session Commands

| Command | Purpose |
|---------|---------|
| `/model <name>` | Switch models mid-session |
| `/new-chat` | Start a fresh chat session |
| `/compress` | Summarize the conversation to save context |
| `/rules` | Create or edit `.cursor/rules` |
| `/quit` | Exit the CLI |

**Important difference from Claude Code**: use `/new-chat` instead of `/clear`, and `/quit` instead of `/exit`.

## Command Line Flags

Common launch patterns:

```bash
agent --force
agent --force --model claude-4.6-sonnet-medium
agent --force --model claude-4.6-sonnet-medium-thinking
```

- `--force`: allow commands unless explicitly denied
- `--model <name>`: choose the startup model
- `--resume [chatId]`: continue a previous chat
- `--print`: non-interactive mode for scripts

## tmux Integration Notes

Cursor CLI is terminal-native and supports slash commands through normal input, which makes it compatible with tmux `send-keys` orchestration.

- Use `agent status` during setup to verify login state
- Use `/new-chat` before a newly assigned task to clear stale context
- Re-send the startup prompt after `/new-chat` so the agent reloads its role instructions
- Use `/quit` for controlled shutdown before relaunching with a new model or CLI type

## Rules Strategy in This Repository

Cursor-specific repository bootstrap is intentionally split:

- `.cursor/rules/cursor-agent-bootstrap.mdc` provides a tiny always-on bootstrap
- `instructions/generated/cursor-*.md` provides the heavy role-specific guidance

This keeps the always-applied rule concise while still giving each tmux agent a full role manual.
