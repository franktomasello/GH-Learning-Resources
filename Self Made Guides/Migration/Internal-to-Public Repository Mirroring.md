# 🪞 Internal-to-Public Repository Mirroring Runbook

> **Complete guide to mirroring internal repositories to public GitHub.com organizations, especially for EMU enterprises that cannot host public repos**

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

## 📚 Resources

- [actions/checkout](https://github.com/actions/checkout)
- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)

---

*Last updated: April 2026*
