# GitHub Code Review

Perform code reviews on local changes before pushing, or review open PRs on GitHub.

## Prerequisites

- Authenticated with GitHub (see `references/authentication.md`)
- Inside a git repository

---

## 1. Reviewing Local Changes (Pre-Push)

```bash
# Staged changes
git diff --staged

# All changes vs main
git diff main...HEAD

# File names only
git diff main...HEAD --name-only

# Stat summary
git diff main...HEAD --stat
```

### Review Strategy

1. Get the big picture:
```bash
git diff main...HEAD --stat
git log main..HEAD --oneline
```

2. Review file by file — use `read_file` on changed files for full context

3. Check for common issues:
```bash
# Debug statements, TODOs
git diff main...HEAD | grep -n "print(\|console\.log\|TODO\|FIXME\|HACK\|XXX\|debugger"

# Large files accidentally staged
git diff main...HEAD --stat | sort -t'|' -k2 -rn | head -10

# Secrets or credential patterns
git diff main...HEAD | grep -in "password\|secret\|api_key\|token.*=\|private_key"

# Merge conflict markers
git diff main...HEAD | grep -n "<<<<<<\|>>>>>>\|======="
```

### Review Output Format

```
## Code Review Summary

### Critical
- **src/auth.py:45** — SQL injection: user input passed directly to query.

### Warnings
- **src/models/user.py:23** — Password stored in plaintext. Use bcrypt or argon2.

### Suggestions
- **src/utils/helpers.py:8** — Duplicates logic in `src/core/utils.py:34`. Consolidate.

### Looks Good
- Clean separation of concerns in the middleware layer
```

---

## 2. Reviewing a Pull Request on GitHub

### View PR Details

```bash
gh pr view 123
gh pr diff 123
gh pr diff 123 --name-only
```

### Check Out PR Locally for Full Review

```bash
git fetch origin pull/123/head:pr-123
git checkout pr-123
# Now you can use read_file, search_files, run tests, etc.
```

### Leave Comments on a PR

```bash
# General comment
gh pr comment 123 --body "Overall looks good, a few suggestions below."

# Inline review comment
HEAD_SHA=$(gh pr view 123 --json headRefOid --jq '.headRefOid')
gh api repos/$OWNER/$REPO/pulls/123/comments --method POST \
  -f body="This could be simplified with a list comprehension." \
  -f path="src/auth/login.py" -f commit_id="$HEAD_SHA" -f line=45 -f side="RIGHT"
```

### Submit a Formal Review

```bash
gh pr review 123 --approve --body "LGTM!"
gh pr review 123 --request-changes --body "See inline comments."
gh pr review 123 --comment --body "Some suggestions, nothing blocking."
```

---

## 3. Review Checklist

- **Correctness** — Does the code do what it claims? Edge cases handled?
- **Security** — No hardcoded secrets, input validation, no injection
- **Code Quality** — Clear naming, DRY, single responsibility
- **Testing** — New code paths tested, happy + error cases
- **Performance** — No N+1 queries, appropriate caching
- **Documentation** — Public APIs documented, "why" comments present

---

## 4. Pre-Push Review Workflow

1. `git diff main...HEAD --stat` — see scope of changes
2. `git diff main...HEAD` — read the full diff
3. For each changed file, use `read_file` for full context
4. Apply the checklist above
5. Present findings in the structured format (Critical / Warnings / Suggestions / Looks Good)
6. If critical issues found, offer to fix them before the user pushes

---

## 5. PR Review Workflow (End-to-End)

1. **Set up environment** — source the auth script or run inline setup
2. **Gather PR context** — `gh pr view 123 && gh pr diff 123 --name-only && gh pr checks 123`
3. **Check out locally** — `git fetch origin pull/123/head:pr-123 && git checkout pr-123`
4. **Read the diff** — `git diff main...HEAD` (file-by-file for large PRs)
5. **Run automated checks** — `python -m pytest`, `ruff check .`, etc.
6. **Apply review checklist**
7. **Post the review to GitHub** — formal review with inline comments
8. **Post a summary comment** — see `references/review-output-template.md`
9. **Clean up** — `git checkout main && git branch -D pr-123`

**Decision: Approve vs Request Changes vs Comment**
- **Approve** — no critical or warning-level issues
- **Request Changes** — any critical or warning-level issue
- **Comment** — observations and suggestions, nothing blocking
