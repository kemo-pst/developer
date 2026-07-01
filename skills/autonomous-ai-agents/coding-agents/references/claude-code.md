# Claude Code — Full Reference

Delegate coding tasks to [Claude Code](https://code.claude.com/docs/en/cli-reference) (Anthropic's autonomous coding agent CLI).

## Prerequisites

- **Install:** `npm install -g @anthropic-ai/claude-code`
- **Auth:** `claude` (browser OAuth), `ANTHROPIC_API_KEY`, `claude auth login --console` (API billing), `claude auth login --sso` (Enterprise)
- **Health:** `claude doctor` | **Version:** `claude --version` (v2.x+) | **Update:** `claude update`

## Two Orchestration Modes

### Mode 1: Print Mode (`-p`) — Non-Interactive (PREFERRED)

```bash
terminal(command="claude -p 'Add error handling to src/' --allowedTools 'Read,Edit' --max-turns 10", workdir="/project", timeout=120)
```

Skips ALL interactive dialogs. Best for: one-shot tasks, CI/CD, structured extraction, piped input.

### Mode 2: Interactive PTY via tmux — Multi-Turn Sessions

```bash
terminal(command="tmux new-session -d -s claude-work -x 140 -y 40")
terminal(command="tmux send-keys -t claude-work 'cd /project && claude' Enter")
terminal(command="sleep 5 && tmux send-keys -t claude-work 'Refactor auth to use JWT' Enter")
terminal(command="sleep 15 && tmux capture-pane -t claude-work -p -S -50")  # monitor
terminal(command="tmux send-keys -t claude-work '/exit' Enter")
```

**Dialog handling:** Trust dialog → press Enter (default "Yes"). Permissions bypass → Down then Enter.

## Key CLI Flags

| Flag | Effect |
|------|--------|
| `-p "query"` | Print mode (exits when done) |
| `-c` / `--continue` | Resume most recent session in this directory |
| `-r <id>` / `--resume <id>` | Resume specific session |
| `--fork-session` | New session ID, keeps history |
| `--max-turns <n>` | Limit agentic loops (print mode only) |
| `--max-budget-usd <n>` | Cap API spend |
| `--model <alias>` | `sonnet`, `opus`, `haiku`, or full name |
| `--effort <level>` | `low`, `medium`, `high`, `max`, `auto` |
| `--allowedTools <tools>` | Whitelist (e.g., `Read,Edit,Bash(git *)`) |
| `--dangerously-skip-permissions` | Auto-approve ALL tool use |
| `--output-format json` | Structured output with session_id, cost, usage |
| `--output-format stream-json` | Real-time streaming |
| `--json-schema '{...}'` | Force structured JSON output |
| `--bare` | Skip hooks/plugins/MCP/CLAUDE.md (fastest) |
| `--worktree [name]` | Isolated git worktree |
| `--from-pr [number]` | Resume session linked to PR |
| `--fallback-model <model>` | Auto-fallback on overload |

## PR Review

```bash
# Quick diff review
git diff main...feature-branch | claude -p 'Review for bugs, security, style' --max-turns 1

# Deep review with worktree
claude -w pr-review  # interactive in worktree

# From PR number
claude -p 'Review this PR' --from-pr 42 --max-turns 10
```

## Parallel Instances

```bash
# Task 1: Fix backend
tmux new-session -d -s task1 && tmux send-keys -t task1 'cd ~/project && claude -p "Fix auth bug" --allowedTools "Read,Edit" --max-turns 10' Enter

# Task 2: Write tests
tmux new-session -d -s task2 && tmux send-keys -t task2 'cd ~/project && claude -p "Write API tests" --allowedTools "Read,Write,Bash" --max-turns 15' Enter

# Monitor
sleep 30 && tmux capture-pane -t task1 -p -S -5 && tmux capture-pane -t task2 -p -S -5
```

## Pitfalls

1. **Interactive mode REQUIRES tmux** — use `capture-pane` for monitoring, `send-keys` for input
2. **`--dangerously-skip-permissions` dialog defaults to "No, exit"** — send Down+Enter
3. **`--max-turns` is print-mode only** — ignored in interactive
4. **Session resumption requires same directory** — `--continue` finds most recent for cwd
5. **Trust dialog appears once per directory** — first-time only
6. **`--bare` skips OAuth** — requires `ANTHROPIC_API_KEY`
7. **Context degrades above 70%** — use `/compact` proactively
8. **Clean up tmux sessions** — `tmux kill-session -t <name>`
