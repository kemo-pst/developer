# OpenCode CLI — Full Reference

Delegate coding tasks to [OpenCode](https://opencode.ai) — a provider-agnostic, open-source AI coding agent with a TUI and CLI.

## Prerequisites

- **Install:** `npm i -g opencode-ai@latest` or `brew install anomalyco/tap/opencode`
- **Auth:** `opencode auth login` or set provider env vars (`OPENROUTER_API_KEY`, etc.)
- **Verify:** `opencode auth list` should show at least one provider
- Git repository for code tasks (recommended)

## One-Shot Tasks

```bash
terminal(command="opencode run 'Add retry logic to API calls and update tests'", workdir="~/project")
```

With attached files:
```bash
terminal(command="opencode run 'Review config for security' -f config.yaml -f .env.example", workdir="~/project")
```

Force model:
```bash
terminal(command="opencode run 'Refactor auth module' --model openrouter/anthropic/claude-sonnet-4", workdir="~/project")
```

## Interactive Sessions (Background)

```bash
terminal(command="opencode", workdir="~/project", background=true, pty=true)
# Returns session_id — send input:
process(action="submit", session_id="<id>", data="Implement OAuth refresh flow")
process(action="poll", session_id="<id>")
process(action="log", session_id="<id>")
# Exit: Ctrl+C
process(action="write", session_id="<id>", data="\x03")
```

**Important:** Do NOT use `/exit` — it opens an agent selector. Use Ctrl+C (`\x03`) or `process(action="kill")`.

## Key Flags

| Flag | Use |
|------|-----|
| `run 'prompt'` | One-shot execution and exit |
| `--continue` / `-c` | Continue the last session |
| `-s <id>` | Continue a specific session |
| `--agent <name>` | Choose agent (build or plan) |
| `--model provider/model` | Force specific model |
| `--format json` | Machine-readable output/events |
| `--file <path>` / `-f` | Attach file(s) |
| `--thinking` | Show model thinking blocks |
| `--variant <level>` | Reasoning effort (high, max, minimal) |
| `--title <name>` | Name the session |

## PR Review

```bash
terminal(command="opencode pr 42", workdir="~/project", pty=true)

# Or in a temp clone:
REVIEW=$(mktemp -d) && git clone https://github.com/user/repo.git $REVIEW
cd $REVIEW && opencode run 'Review this PR vs main. Report bugs, security risks, test gaps.'
```

## Parallel Work Pattern

```bash
terminal(command="opencode run 'Fix issue #101 and commit'", workdir="/tmp/issue-101", background=true, pty=true)
terminal(command="opencode run 'Add parser tests and commit'", workdir="/tmp/issue-102", background=true, pty=true)
process(action="list")
```

## Session & Cost Management

```bash
opencode session list
opencode stats
opencode stats --days 7 --models anthropic/claude-sonnet-4
```

## Pitfalls

1. **Interactive sessions need `pty=true`** — `opencode run` does NOT need pty
2. **`/exit` is NOT valid** — opens agent selector. Use Ctrl+C
3. **PATH mismatch** — check `which -a opencode` if behavior differs
4. **If stuck, inspect logs before killing** — `process(action="log", session_id="<id>")`
5. **Avoid sharing one workdir across parallel sessions**
6. **Enter may need to be pressed twice** in TUI (once to finalize, once to send)

## Verification

```bash
opencode run 'Respond with exactly: OPENCODE_SMOKE_OK'
```

## Rules

1. Prefer `opencode run` for one-shot automation
2. Use interactive background mode only when iteration is needed
3. Always scope to a single repo/workdir
4. For long tasks, provide progress updates from `process` logs
5. Report concrete outcomes (files changed, tests, remaining risks)
6. Exit with Ctrl+C or kill, never `/exit`
