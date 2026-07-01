---
name: coding-agents
description: "Delegate coding to external CLI coding agents: Claude Code, Codex, OpenCode. Use when the user asks to delegate implementation, refactoring, or PR review to an autonomous coding CLI, or when a task would benefit from an isolated coding agent with full file/shell access."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [coding-agents, delegation, claude-code, codex, opencode, autonomous, refactoring]
    related_skills: [claude-code, codex, opencode]
---

# Coding Agent Delegation

Delegate coding tasks to external CLI coding agents. All three supported agents (Claude Code, Codex, OpenCode) can read/write files, run shell commands, manage git workflows, and autonomously implement features.

## Choosing an Agent

| Agent | Provider | Best For | Install |
|-------|----------|----------|---------|
| **Claude Code** | Anthropic | General coding, PR review, refactoring | `npm i -g @anthropic-ai/claude-code` |
| **Codex** | OpenAI | One-shot tasks, batch issue fixing | `npm i -g @openai/codex` |
| **OpenCode** | Any provider | Provider-agnostic, model-flexible | `npm i -g opencode-ai@latest` |

## Common Patterns (All Agents)

### One-Shot Tasks
```
agent run 'Implement feature X and add tests'
```

### Interactive Sessions
```
agent  # launch TUI, send tasks via process(submit)
```

### PR Review
```
agent review PR_NUMBER  # or: git diff main...HEAD | agent 'Review this diff'
```

### Parallel Work
```
# Launch multiple agents in different workdirs/worktrees
agent run 'Fix issue A' → /tmp/work-a
agent run 'Fix issue B' → /tmp/work-b
```

---

## Claude Code

→ Full guide: `references/claude-code.md`

**Quick start:**
```bash
# Print mode (preferred for most tasks)
claude -p 'Add error handling to src/' --allowedTools 'Read,Edit' --max-turns 10

# Interactive via tmux
tmux new-session -d -s claude-work -x 140 -y 40
tmux send-keys -t claude-work 'cd /project && claude' Enter
tmux capture-pane -t claude-work -p -S -50  # monitor
```

**Key flags:** `-p` (print), `--max-turns`, `--allowedTools`, `--model`, `--dangerously-skip-permissions`, `--worktree`, `--from-pr`

---

## Codex

→ Full guide: `references/codex.md`

**Quick start:**
```bash
codex exec 'Add dark mode toggle to settings'                    # one-shot
codex exec --full-auto 'Refactor the auth module'                # background + auto-approve
codex review --base origin/main                                  # PR review
```

**Key flags:** `exec`, `--full-auto`, `--yolo`, `--sandbox danger-full-access`

**Caveat:** Requires `pty=true` and a git repo. May fail in gateway/service contexts due to sandboxing.

---

## OpenCode

→ Full guide: `references/opencode.md`

**Quick start:**
```bash
opencode run 'Add retry logic to API calls'                       # one-shot
opencode run 'Refactor auth' --model openrouter/anthropic/claude-sonnet-4  # specific model
opencode pr 42                                                   # PR review
```

**Key flags:** `run`, `--continue`, `--model`, `--agent`, `--thinking`, `--file`

**Caveat:** Use Ctrl+C (`\x03`) to exit, NOT `/exit`. Use `pty=true` for interactive sessions.

---

## Rules (All Agents)

1. **Always set `workdir`** — keep the agent focused on the right project
2. **Set `--max-turns` / bounds in one-shot mode** — prevents runaway loops
3. **Monitor background sessions** — use `process(action='poll'|'log')`
4. **Clean up** — kill background sessions, remove temp worktrees when done
5. **Report results** — summarize what the agent did and what changed
6. **Don't interfere** — let the agent work; check progress via logs instead of killing
7. **Git repo required** — all agents need a git repo (use `mktemp -d && git init` for scratch)
