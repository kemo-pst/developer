# GitHub Pull Request Workflow

Complete guide for managing the PR lifecycle. Each section shows the `gh` way first, then the `git` + `curl` fallback.

## Prerequisites

- Authenticated with GitHub (see `references/authentication.md`)
- Inside a git repository with a GitHub remote

---

## 1. Branch Creation

```bash
git fetch origin
git checkout main && git pull origin main
git checkout -b feat/add-user-authentication
```

Branch naming: `feat/`, `fix/`, `refactor/`, `docs/`, `ci/`

## 2. Making Commits

```bash
git add src/auth.py tests/test_auth.py
git commit -m "feat: add JWT-based user authentication

- Add login/register endpoints
- Add User model with password hashing
- Add auth middleware for protected routes"
```

See `references/conventional-commits.md` for the full commit message format.

## 3. Pushing and Creating a PR

```bash
git push -u origin HEAD

# With gh
gh pr create --title "feat: add JWT authentication" --body "## Summary\n..." --reviewer user1,user2

# With curl
BRANCH=$(git branch --show-current)
curl -s -X POST -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls \
  -d "{\"title\": \"feat: ...\", \"body\": \"...\", \"head\": \"$BRANCH\", \"base\": \"main\"}"
```

Options: `--draft`, `--reviewer user1,user2`, `--label "enhancement"`, `--base develop`

## 4. Monitoring CI Status

```bash
# With gh
gh pr checks
gh pr checks --watch

# With curl — get combined status
SHA=$(git rev-parse HEAD)
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/status \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
print(f\"Overall: {data['state']}\")
for s in data.get('statuses', []):
    print(f\"  {s['context']}: {s['state']} - {s.get('description', '')}\")"
```

See `references/ci-troubleshooting.md` for auto-fix loops and CI failure diagnosis.

## 5. Auto-Fixing CI Failures

1. Get failure details: `gh run view <RUN_ID> --log-failed`
2. Read failure logs → understand the error
3. Fix with `patch`/`write_file`
4. `git add . && git commit -m "fix: ..." && git push`
5. Re-check CI status
6. Repeat if still failing (up to 3 attempts, then ask the user)

## 6. Merging

```bash
# With gh
gh pr merge --squash --delete-branch
gh pr merge --auto --squash --delete-branch  # enable auto-merge

# With curl
curl -s -X PUT -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER/merge \
  -d '{"merge_method": "squash", "commit_title": "feat: add user authentication (#123)"}'
```

Merge methods: `"merge"`, `"squash"`, `"rebase"`

## 7. Complete Workflow Example

```bash
git checkout main && git pull origin main
git checkout -b fix/login-redirect-bug
# Agent makes code changes with file tools
git add src/auth/login.py tests/test_login.py
git commit -m "fix: correct redirect URL after login"
git push -u origin HEAD
gh pr create --title "fix: correct redirect URL after login" --body "..."
gh pr checks --watch
gh pr merge --squash --delete-branch
```

## Useful PR Commands Reference

| Action | gh | git + curl |
|--------|-----|-----------|
| List my PRs | `gh pr list --author @me` | `curl .../pulls?state=open` |
| View PR diff | `gh pr diff` | `git diff main...HEAD` |
| Add comment | `gh pr comment N --body "..."` | `curl POST .../issues/N/comments` |
| Request review | `gh pr edit N --add-reviewer user` | `curl POST .../pulls/N/requested_reviewers` |
| Close PR | `gh pr close N` | `curl PATCH .../pulls/N -d '{"state":"closed"}'` |
| Check out PR | `gh pr checkout N` | `git fetch origin pull/N/head:pr-N && git checkout pr-N` |
