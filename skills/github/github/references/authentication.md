# GitHub Authentication Setup

This skill sets up authentication so the agent can work with GitHub repositories, PRs, issues, and CI. It covers two paths:

- **`git` (always available)** — uses HTTPS personal access tokens or SSH keys
- **`gh` CLI (if installed)** — richer GitHub API access with a simpler auth flow

## Detection Flow

```bash
git --version
gh --version 2>/dev/null || echo "gh not installed"
gh auth status 2>/dev/null || echo "gh not authenticated"
git config --global credential.helper 2>/dev/null || echo "no git credential helper"
```

**Decision tree:**
1. If `gh auth status` shows authenticated → you're good, use `gh` for everything
2. If `gh` is installed but not authenticated → use "gh auth" method below
3. If `gh` is not installed → use "git-only" method below (no sudo needed)

---

## Method 1: Git-Only Authentication (No gh, No sudo)

### Option A: HTTPS with Personal Access Token (Recommended)

**Step 1: Create a personal access token**
Tell the user to go to: **https://github.com/settings/tokens**
- Click "Generate new token (classic)"
- Give it a name like "hermes-agent"
- Select scopes: `repo`, `workflow`, `read:org` (if working with org repos)
- Set expiration (90 days is a good default)
- Copy the token — it won't be shown again

**Step 2: Configure git to store the token**
```bash
git config --global credential.helper store
# Now do a test operation that triggers auth:
git ls-remote https://github.com/<their-username>/<any-repo>.git
# Username: <their-github-username>
# Password: <paste the personal access token, NOT their GitHub password>
```

Alternative: cache helper (credentials expire from memory):
```bash
git config --global credential.helper 'cache --timeout=28800'
```

Alternative: set the token directly in the remote URL (per-repo):
```bash
git remote set-url origin https://<username>:<token>@github.com/<owner>/<repo>.git
```

**Step 3: Configure git identity**
```bash
git config --global user.name "Their Name"
git config --global user.email "their-email@example.com"
```

**Step 4: Verify**
```bash
git ls-remote https://github.com/<their-username>/<any-repo>.git
git config --global user.name
git config --global user.email
```

### Option B: SSH Key Authentication

```bash
# Check for existing keys
ls -la ~/.ssh/id_*.pub 2>/dev/null || echo "No SSH keys found"

# Generate a key if needed
ssh-keygen -t ed25519 -C "their-email@example.com" -f ~/.ssh/id_ed25519 -N ""
cat ~/.ssh/id_ed25519.pub  # → add to https://github.com/settings/keys

# Test
ssh -T git@github.com

# Configure git to use SSH for GitHub
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

---

## Method 2: gh CLI Authentication

### Interactive Browser Login (Desktop)
```bash
gh auth login
# Select: GitHub.com → HTTPS → Authenticate via browser
```

### Token-Based Login (Headless / SSH Servers)
```bash
echo "<THEIR_TOKEN>" | gh auth login --with-token
gh auth setup-git
```

### Verify
```bash
gh auth status
```

---

## Using the GitHub API Without gh

```bash
export GITHUB_TOKEN="<token>"
curl -s -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/user
```

Extract from git credentials:
```bash
grep "github.com" ~/.git-credentials 2>/dev/null | head -1 | sed 's|https://[^:]*:\([^@]*\)@.*|\1|'
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `git push` asks for password | Use a personal access token as the password |
| `remote: Permission to X denied` | Token may lack `repo` scope — regenerate |
| `fatal: Authentication Failed` | Stale cached credentials — `git credential reject` then re-auth |
| `ssh: connect to host github.com port 22: Connection refused` | Try SSH over HTTPS port 443 via `ssh.github.com` |
| Credentials not persisting | Check `git config --global credential.helper` |
| Multiple GitHub accounts | Use SSH with different keys per host alias |
| `gh: command not found` + no sudo | Use git-only Method 1 above |
