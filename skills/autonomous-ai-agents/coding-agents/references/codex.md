# Codex CLI — Full Reference

Delegate coding tasks to [Codex](https://github.com/openai/codex) (OpenAI's autonomous coding agent CLI).

## Prerequisites

- **Install:** `npm install -g @openai/codex`
- **Auth:** `OPENAI_API_KEY` or Codex OAuth (`codex auth login`)
- **Requires a git repository** — Codex refuses to run outside one

## One-Shot Tasks

```bash
terminal(command="codex exec 'Add dark mode toggle to settings'", workdir="~/project", pty=true)
```

For scratch work (Codex needs a git repo):
```bash
terminal(command="cd $(mktemp -d) && git init && codex exec 'Build a snake game in Python'", pty=true)
```

## Background Mode (Long Tasks)

```bash
terminal(command="codex exec --full-auto 'Refactor the auth module'", workdir="~/project", background=true, pty=true)
# Returns session_id — monitor with:
process(action="poll", session_id="<id>")
process(action="log", session_id="<id>")
process(action="submit", session_id="<id>", data="yes")  # send input
process(action="kill", session_id="<id>")  # kill
```

## Key Flags

| Flag | Effect |
|------|--------|
| `exec "prompt"` | One-shot execution, exits when done |
| `--full-auto` | Sandboxed but auto-approves file changes |
| `--yolo` | No sandbox, no approvals (fastest, most dangerous) |
| `--sandbox danger-full-access` | Disable Codex sandbox (for gateway contexts) |

## Gateway/Service Context Caveat

Codex sandboxing may fail in Hermes gateway contexts (bubblewrap/user-namespace errors). Use:
```bash
codex exec --sandbox danger-full-access "<task>"
```

## PR Reviews

```bash
# Clone to temp for safe review
REVIEW=$(mktemp -d) && git clone https://github.com/user/repo.git $REVIEW
cd $REVIEW && gh pr checkout 42 && codex review --base origin/main
```

## Parallel Issue Fixing with Worktrees

```bash
git worktree add -b fix/issue-78 /tmp/issue-78 main
git worktree add -b fix/issue-99 /tmp/issue-99 main
terminal(command="codex --yolo exec 'Fix issue #78. Commit when done.'", workdir="/tmp/issue-78", background=true, pty=true)
terminal(command="codex --yolo exec 'Fix issue #99. Commit when done.'", workdir="/tmp/issue-99", background=true, pty=true)
process(action="list")  # monitor both
```

## Batch PR Reviews

```bash
git fetch origin '+refs/pull/*/head:refs/remotes/origin/pr/*'
terminal(command="codex exec 'Review PR #86. git diff origin/main...origin/pr/86'", workdir="~/project", background=true, pty=true)
terminal(command="codex exec 'Review PR #87. git diff origin/main...origin/pr/87'", workdir="~/project", background=true, pty=true)
```

## Rules

1. **Always use `pty=true`** — Codex is an interactive terminal app
2. **Git repo required** — use `mktemp -d && git init` for scratch
3. **Use `exec` for one-shots** — runs and exits cleanly
4. **`--full-auto` for building** — auto-approves within sandbox
5. **Background for long tasks** — use `background=true` + `process` tool
6. **Don't interfere** — monitor with `poll`/`log`, be patient
7. **Parallel is fine** — run multiple Codex processes for batch work
