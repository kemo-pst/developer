---
name: github
description: "GitHub operations: auth, issues, PRs, code review, repo management, releases, Actions. Use when the user asks to work with GitHub in any way — manage issues, review PRs, create repos, push code, manage releases, triage, or interact with the GitHub API."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [github, git, issues, pull-requests, code-review, repos, releases, actions, gh-cli]
    related_skills: [github-auth, github-code-review, github-issues, github-pr-workflow, github-repo-management]
---

# GitHub Operations

This is the umbrella skill for all GitHub operations. It covers authentication, issue management, PR lifecycle, code review, repository management, releases, and GitHub Actions. Each section shows the `gh` CLI way first, then the `git` + `curl` fallback.

## Detection & Setup

```bash
# Check what's available
git --version
gh --version 2>/dev/null || echo "gh not installed"
gh auth status 2>/dev/null || echo "gh not authenticated"

# Determine auth method
if command -v gh &>/dev/null && gh auth status &>/dev/null; then
  AUTH="gh"
else
  AUTH="git"
  if [ -z "$GITHUB_TOKEN" ]; then
    if _hermes_env="${HERMES_HOME:-$HOME/.hermes}/.env"; [ -f "$_hermes_env" ] && grep -q "^GITHUB_TOKEN=" "$_hermes_env"; then
      export GITHUB_TOKEN=$(grep "^GITHUB_TOKEN=" "$_hermes_env" | head -1 | cut -d= -f2 | tr -d '\n\r')
    elif grep -q "github.com" ~/.git-credentials 2>/dev/null; then
      export GITHUB_TOKEN=$(grep "github.com" ~/.git-credentials | head -1 | sed 's|https://[^:]*:\([^@]*\)@.*|\1|')
    fi
  fi
fi

# Extract owner/repo from git remote (when inside a repo)
REMOTE_URL=$(git remote get-url origin)
OWNER_REPO=$(echo "$REMOTE_URL" | sed -E 's|.*github\.com[:/]||; s|\.git$||')
OWNER=$(echo "$OWNER_REPO" | cut -d/ -f1)
REPO=$(echo "$OWNER_REPO" | cut -d/ -f2)
```

---

## 1. Authentication

→ Full guide: `references/authentication.md` (HTTPS tokens, SSH keys, gh CLI login, API without gh, troubleshooting)

**Quick path:**
1. If `gh auth status` shows authenticated → use `gh` for everything
2. If `gh` installed but not authenticated → `gh auth login` or `echo "$TOKEN" | gh auth login --with-token`
3. If no `gh` → use git credential helper with a personal access token

---

## 2. Issues

→ Full guide: `references/issues.md` (create, triage, label, assign, close, bulk operations)

```bash
gh issue list --state open
gh issue create --title "Bug: ..." --body "..." --label "bug"
gh issue view 42 && gh issue edit 42 --add-label "priority:high" && gh issue close 42
```

---

## 3. Pull Request Lifecycle

→ Full guide: `references/pr-workflow.md` (branch, commit, push, create, CI monitoring, merge)

```bash
git checkout -b fix/the-bug && git add . && git commit -m "fix: resolve the bug"
git push -u origin HEAD
gh pr create --title "fix: resolve the bug" --body "## Summary\n..."
gh pr checks --watch && gh pr merge --squash --delete-branch
```

---

## 4. Code Review

→ Full guide: `references/code-review.md` (pre-push local review, PR review, inline comments, checklist)

```bash
git diff main...HEAD --stat && git diff main...HEAD  # local review
gh pr checkout 123 && git diff main...HEAD           # PR review
gh pr review 123 --approve --body "LGTM"
```

---

## 5. Repository Management

→ Full guide: `references/repo-management.md` (clone, create, fork, branch protection, secrets, releases, Actions, gists)

```bash
gh repo clone owner/repo
gh repo create my-project --public --clone
gh repo fork owner/repo --clone
gh release create v1.0.0 --generate-notes
gh workflow list && gh run list --limit 10
```

---

## Quick Reference Table

| Action | gh | git + curl |
|--------|-----|-----------|
| Auth setup | `gh auth login` | `git config credential.helper store` + token |
| List issues | `gh issue list` | `curl GET /repos/{o}/{r}/issues` |
| Create repo | `gh repo create name` | `curl POST /user/repos` |
| Create PR | `gh pr create ...` | `curl POST /repos/{o}/{r}/pulls` |
| Merge PR | `gh pr merge --squash` | `curl PUT /repos/{o}/{r}/pulls/{n}/merge` |
| Create release | `gh release create v1.0` | `curl POST /repos/{o}/{r}/releases` |
| List workflows | `gh workflow list` | `curl GET /repos/{o}/{r}/actions/workflows` |
| Rerun CI | `gh run rerun ID` | `curl POST /repos/{o}/{r}/actions/runs/{id}/rerun` |
