# 🪞 Internal-to-Public Repository Mirroring Runbook

> **Complete guide to mirroring internal repositories to public GitHub.com organizations, especially for EMU enterprises that cannot host public repos**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create public org:** Create a new organization on github.com (not under EMU enterprise)
- **Store mirror token:** `Internal Repo → Settings → Secrets and variables → Actions → New secret → MIRROR_TOKEN`
- **Add workflow:** Create `.github/workflows/mirror-to-public.yml` in internal repo
- **Enable secret scanning:** `Internal Repo → Settings → Security → Advanced Security → Secret Protection → Enable`
- **Add branch protection:** `Internal Repo → Settings → Rules → Rulesets → New branch ruleset → Require PR reviews`

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Separate public organization on github.com (not under EMU enterprise) | ☐ |
| Target public repository created in the public org | ☐ |
| Authentication token (PAT, deploy key, or GitHub App) for the public repo | ☐ |
| GitHub Actions enabled on the internal (source) repository | ☐ |
| Secret scanning enabled on the internal repo | ☐ |

---

## 📋 Overview

Enterprise Managed User (EMU) enterprises do not support public repositories. Organizations that need to publish open-source code or share projects publicly must mirror approved content from their internal EMU enterprise to a separate public organization on github.com.

| Approach | Complexity | Best For |
|----------|-----------|----------|
| **Separate public org (manual)** | Low | Occasional publishing, small teams |
| **Automated mirror via GitHub Actions** | Medium | Frequent releases, CI/CD pipelines |
| **GitHub App-based automation** | Higher | Production-grade, auditable mirroring |

---

## 1️⃣ Option 1 — Separate Public Organization (Recommended Starting Point)

*Create a non-EMU organization on github.com dedicated to open-source publishing*

### Setup

1. Create a new organization on github.com (not under the EMU enterprise)
2. Ensure the org is **not** managed by your EMU IdP — it should use standard GitHub.com authentication
3. Designate maintainers who have accounts in both the EMU enterprise and the public org
4. Manually push approved code to the public org repositories

> 💡 **Tip:** This approach gives you full control over what is published. Pair it with an internal approval process (e.g., a PR-based review gate) before any code is pushed to the public org.

---

## 2️⃣ Option 2 — Automated Mirror via GitHub Actions

*Push-triggered workflow that mirrors commits from an internal repo to a public repo*

### Example Workflow

Create this file in your **internal** (source) repository:

```yaml
# .github/workflows/mirror-to-public.yml
name: Mirror to Public Repository

on:
  push:
    branches:
      - main

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for accurate mirroring

      - name: Push to public repository
        run: |
          git remote add public https://x-access-token:${{ secrets.MIRROR_TOKEN }}@github.com/your-public-org/your-public-repo.git
          git push public main --force
        env:
          MIRROR_TOKEN: ${{ secrets.MIRROR_TOKEN }}
```

### Setup Steps

1. Create the target public repository in your public org
2. Generate an authentication token (see Section 3 below)
3. Store the token as a repository secret named `MIRROR_TOKEN`:

**Navigation:**

```
Internal Repository → Settings → Secrets and variables → Actions
  → New repository secret → Name: MIRROR_TOKEN → Paste token value
```

4. Push a commit to `main` to trigger the workflow

> ✅ **Result:** Every push to `main` in the internal repo will automatically mirror to the public repo.

> ⚠️ **Warning:** The `--force` flag overwrites the public repo's history. Do not make commits directly on the public repo — treat it as read-only.

---

## 3️⃣ Authentication Options for Mirroring

| Method | Security | Scope | Best For |
|--------|----------|-------|----------|
| **Personal Access Token (PAT)** | Basic | Tied to a user account | Quick setup, testing |
| **Deploy Key** | Better | Tied to a single repository | Repo-scoped access without a user account |
| **GitHub App Token** | Best | Scoped to specific permissions and repos | Production-grade, auditable, not tied to a person |

### A) Personal Access Token (PAT)

**Navigation:**

```
GitHub.com (public org account) → Profile Picture → Settings
  → Developer settings → Personal access tokens → Fine-grained tokens
    → Generate new token
```

| Setting | Value |
|---------|-------|
| **Repository access** | Only select repositories → choose the public target repo |
| **Permissions** | Contents: Read and write |

### B) Deploy Key

**Navigation:**

```
Public Target Repository → Settings → Deploy keys
  → Add deploy key → Paste public SSH key → Check "Allow write access"
```

> 💡 **Tip:** Generate a dedicated SSH key pair for mirroring. Store the private key as a repository secret in the internal repo.

### C) GitHub App (Most Secure)

1. Create a GitHub App in the public org with `Contents: Read and write` permission
2. Install the App on the target public repository
3. Use the App's private key to generate short-lived installation tokens in the workflow

> 💡 **Tip:** GitHub App tokens expire after 1 hour, reducing risk if a token is exposed. This is the recommended approach for production mirroring.

---

## 4️⃣ Selective Mirroring

*Mirror only what should be public — not everything*

| Technique | How |
|-----------|-----|
| **Mirror only a release branch** | Change the workflow trigger to a specific branch (e.g., `release/public`) |
| **Exclude files with .gitignore** | Add internal-only files to `.gitignore` before they reach the mirror branch |
| **Use git filter-repo** | Strip files, directories, or secrets from history before pushing to public |
| **Dedicated mirror branch** | Maintain a `mirror` branch that only contains approved content; trigger the workflow on that branch |

### Example: Mirror Only a Release Branch

```yaml
on:
  push:
    branches:
      - release/public  # Only mirror when this branch is updated
```

---

## 5️⃣ Security Controls

> ⚠️ **Important:** Mirroring can accidentally expose secrets, internal paths, or proprietary code. Apply these controls before enabling any mirror.

| Control | How to Enable |
|---------|---------------|
| **Secret scanning on source repo** | Enable Secret Protection on the internal repo to catch leaked secrets before they reach the mirror |
| **Push protection** | Block secrets from being committed to the mirror branch |
| **Required PR approval for mirror branch** | Use branch protection rules or rulesets to require review before merging to the mirror branch |
| **Audit logs** | Review enterprise audit logs for mirror workflow runs and token usage |

### Enable Secret Scanning on the Source Repo

**Navigation:**

```
Internal Repository → Settings → Security (sidebar) → Advanced Security
  → Secret Protection → Enable
    → Push Protection → Enable
```

### Require PR Approval for the Mirror Branch

**Navigation:**

```
Internal Repository → Settings → Rules → Rulesets
  → New ruleset → Branch ruleset
    → Target: release/public (or main)
    → Require pull request reviews before merging → Enable
```

> ✅ **Result:** No code reaches the public mirror without at least one reviewer approving it.

---

## ❓ Common Questions & Troubleshooting

### Q: The mirror workflow is failing with authentication errors. What should I check?
**A:** Verify that the PAT, deploy key, or GitHub App token stored as a secret has not expired. Check that the secret name in the workflow matches the actual secret name in Settings > Secrets and variables > Actions (e.g., `MIRROR_TOKEN`). For PATs, confirm the token still has Contents: Read and Write permission on the target public repository. Regenerate and re-store the credential if needed.

---

### Q: Sensitive data was accidentally pushed to the public mirror. What should we do immediately?
**A:** Treat this as a security incident. First, immediately revoke and rotate any exposed credentials (API keys, tokens, passwords). Then use `git filter-repo` to remove the sensitive data from the public repo's history and force-push the cleaned history. Notify your security team and assess the exposure window. Consider temporarily deleting the public repo if the exposure is severe, and re-mirror after cleanup.

---

### Q: The public mirror only shows the latest commit instead of the full history. What is wrong?
**A:** The `actions/checkout` step in the workflow is using the default shallow clone (`fetch-depth: 1`). Set `fetch-depth: 0` in the checkout step to fetch the full commit history before pushing to the mirror. Without full history, only the most recent commit is pushed.

---

### Q: How do I stop mirroring to the public repository?
**A:** Remove or disable the GitHub Actions mirror workflow in the internal repository (delete the `.github/workflows/mirror-to-public.yml` file or disable the workflow in the Actions tab). Also delete the deploy key from the public repository (or revoke the PAT/App token) to prevent any residual access. The public repo will remain in its current state but will no longer receive updates.

---

### Q: Can I mirror only specific files or directories instead of the entire repository?
**A:** Yes. Use a dedicated mirror branch that contains only the approved content, and configure the workflow to push only that branch. Alternatively, use `git filter-repo` in the workflow to strip internal-only files before pushing. You can also maintain a `.gitignore` on the mirror branch that excludes internal files, though this only affects new commits.

---

### Q: The mirror workflow runs but nothing changes in the public repo. What could be the issue?
**A:** If there are no new commits on the source branch since the last mirror run, `git push` will report "Everything up-to-date" and no changes will appear. Also verify the workflow trigger branch matches the branch you are committing to. Check the workflow run logs in the Actions tab for any error messages or skipped steps.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| EMU Dual Presence (Enterprise + Open Source) | `Identity/EMU Dual Presence (Enterprise + Open Source).md` |
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| GitHub App for CI/CD (No Seat Cost) | `Actions/GitHub App for CI-CD (No Seat Cost).md` |

---

## 📚 Resources

- [actions/checkout](https://github.com/actions/checkout)
- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)

---

*Last updated: April 2026*
