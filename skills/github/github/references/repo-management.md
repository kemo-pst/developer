# GitHub Repository Management

Create, clone, fork, configure, and manage GitHub repositories. Each section shows `gh` first, then the `git` + `curl` fallback.

## Prerequisites

- Authenticated with GitHub (see `references/authentication.md`)

---

## 1. Cloning Repositories

```bash
git clone https://github.com/owner/repo-name.git
git clone --depth 1 https://github.com/owner/repo-name.git  # shallow
git clone --branch develop https://github.com/owner/repo-name.git
gh repo clone owner/repo-name  # gh shorthand
```

## 2. Creating Repositories

```bash
# With gh
gh repo create my-new-project --public --clone
gh repo create my-new-project --private --description "A tool" --license mit --clone
gh repo create my-org/my-new-project --public --clone  # under org
gh repo create my-new-app --template owner/template-repo --public --clone  # from template

# With curl
curl -s -X POST -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/repos \
  -d '{"name": "my-new-project", "description": "A tool", "private": false, "auto_init": true}'
```

## 3. Forking Repositories

```bash
gh repo fork owner/repo-name --clone

# With curl
curl -s -X POST -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/owner/repo-name/forks
sleep 3
git clone https://github.com/$GH_USER/repo-name.git
cd repo-name
git remote add upstream https://github.com/owner/repo-name.git
```

### Keeping a Fork in Sync

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
gh repo sync $GH_USER/repo-name  # gh shorthand
```

## 4. Repository Information

```bash
gh repo view owner/repo-name
gh repo list --limit 20
gh search repos "machine learning" --language python --sort stars
```

## 5. Repository Settings

```bash
gh repo edit --description "Updated" --visibility public
gh repo edit --default-branch main
gh repo edit --add-topic "machine-learning,python"
gh repo edit --enable-auto-merge
```

## 6. Branch Protection

```bash
curl -s -X PUT -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/branches/main/protection \
  -d '{
    "required_status_checks": {"strict": true, "contexts": ["ci/test"]},
    "enforce_admins": false,
    "required_pull_request_reviews": {"required_approving_review_count": 1},
    "restrictions": null
  }'
```

## 7. Secrets Management (GitHub Actions)

```bash
gh secret set API_KEY --body "your-secret-value"
gh secret set SSH_KEY < ~/.ssh/id_rsa
gh secret list
gh secret delete API_KEY
```

## 8. Releases

```bash
gh release create v1.0.0 --title "v1.0.0" --generate-notes
gh release create v2.0.0-rc1 --draft --prerelease --generate-notes
gh release list
gh release download v1.0.0 --dir ./downloads
```

## 9. GitHub Actions Workflows

```bash
gh workflow list
gh run list --limit 10
gh run view <RUN_ID> --log-failed
gh run rerun <RUN_ID>
gh run rerun <RUN_ID> --failed
gh workflow run ci.yml --ref main
gh workflow run deploy.yml -f environment=staging
```

## 10. Gists

```bash
gh gist create script.py --public --desc "Useful script"
gh gist list
```

## Quick Reference Table

| Action | gh | git + curl |
|--------|-----|-----------|
| Clone | `gh repo clone o/r` | `git clone https://github.com/o/r.git` |
| Create repo | `gh repo create name --public` | `curl POST /user/repos` |
| Fork | `gh repo fork o/r --clone` | `curl POST /repos/o/r/forks` + `git clone` |
| Repo info | `gh repo view o/r` | `curl GET /repos/o/r` |
| Edit settings | `gh repo edit --...` | `curl PATCH /repos/o/r` |
| Create release | `gh release create v1.0` | `curl POST /repos/o/r/releases` |
| List workflows | `gh workflow list` | `curl GET /repos/o/r/actions/workflows` |
| Rerun CI | `gh run rerun ID` | `curl POST /repos/o/r/actions/runs/ID/rerun` |
| Set secret | `gh secret set KEY` | `curl PUT /repos/o/r/actions/secrets/KEY` |
