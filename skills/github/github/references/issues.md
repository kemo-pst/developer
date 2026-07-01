# GitHub Issues Management

Create, search, triage, and manage GitHub issues. Each section shows `gh` first, then the `curl` fallback.

## Prerequisites

- Authenticated with GitHub (see `references/authentication.md`)
- Inside a git repo with a GitHub remote, or specify the repo explicitly

---

## 1. Viewing Issues

```bash
# With gh
gh issue list
gh issue list --state open --label "bug"
gh issue list --assignee @me
gh issue list --search "authentication error" --state all
gh issue view 42

# With curl
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?state=open&per_page=20" \
  | python3 -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:
        labels = ', '.join(l['name'] for l in i['labels'])
        print(f\"#{i['number']:5}  {i['state']:6}  {labels:30}  {i['title']}\")"
```

## 2. Creating Issues

```bash
# With gh
gh issue create --title "Login redirect ignores ?next= parameter" \
  --body "## Description\n..." --label "bug,backend" --assignee "username"

# With curl
curl -s -X POST -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues \
  -d '{"title": "...", "body": "...", "labels": ["bug"], "assignees": ["username"]}'
```

### Templates

- Bug reports: see `templates/bug-report.md`
- Feature requests: see `templates/feature-request.md`

## 3. Managing Issues

```bash
# Labels
gh issue edit 42 --add-label "priority:high,bug"
gh issue edit 42 --remove-label "needs-triage"

# Assignment
gh issue edit 42 --add-assignee username

# Commenting
gh issue comment 42 --body "Investigated — root cause is in auth middleware."

# Closing and reopening
gh issue close 42
gh issue close 42 --reason "not planned"
gh issue reopen 42

# Linking to PRs (auto-close on merge)
# Include "Closes #42" / "Fixes #42" / "Resolves #42" in PR body
```

## 4. Issue Triage Workflow

1. List untriaged issues: `gh issue list --label "needs-triage" --state open`
2. Read and categorize each issue
3. Apply labels and priority
4. Assign if the owner is clear
5. Comment with triage notes if needed

## 5. Bulk Operations

```bash
# Close all issues with a specific label
gh issue list --label "wontfix" --json number --jq '.[].number' | \
  xargs -I {} gh issue close {} --reason "not planned"
```

## Quick Reference Table

| Action | gh | curl endpoint |
|--------|-----|--------------|
| List issues | `gh issue list` | `GET /repos/{o}/{r}/issues` |
| View issue | `gh issue view N` | `GET /repos/{o}/{r}/issues/N` |
| Create issue | `gh issue create ...` | `POST /repos/{o}/{r}/issues` |
| Add labels | `gh issue edit N --add-label ...` | `POST /repos/{o}/{r}/issues/N/labels` |
| Assign | `gh issue edit N --add-assignee ...` | `POST /repos/{o}/{r}/issues/N/assignees` |
| Comment | `gh issue comment N --body ...` | `POST /repos/{o}/{r}/issues/N/comments` |
| Close | `gh issue close N` | `PATCH /repos/{o}/{r}/issues/N` |
| Search | `gh issue list --search "..."` | `GET /search/issues?q=...` |
