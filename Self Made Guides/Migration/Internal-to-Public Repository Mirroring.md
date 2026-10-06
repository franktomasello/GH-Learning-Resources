# 🪞 GitHub Internal-to-Public Repository Mirroring Runbook

> **Complete guide to mirroring internal repositories to public GitHub.com organizations, especially for EMU enterprises that cannot host public repos**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Option 1 — Separate Public Organization (Recommended Starting Point)](#1️⃣-option-1--separate-public-organization-recommended-starting-point)
- [2️⃣ Option 2 — Automated Mirror via GitHub Actions](#2️⃣-option-2--automated-mirror-via-github-actions)
- [3️⃣ Authentication Options for Mirroring](#3️⃣-authentication-options-for-mirroring)
- [4️⃣ Selective Mirroring](#4️⃣-selective-mirroring)
- [5️⃣ Security Controls](#5️⃣-security-controls)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Public org:** create a separate organization on GitHub.com that is **not** in the EMU enterprise
- **Auth (recommended):** a GitHub App owned by the public org, installed on the target repo → store `PUBLIC_APP_CLIENT_ID` (variable) and `PUBLIC_APP_PRIVATE_KEY` (secret) in the internal repo
- **Approval gate:** `Internal Repo → Settings → Environments → New environment` → `public-release` → **Required reviewers** → **Save protection rules**
- **Workflow:** `.github/workflows/mirror-to-public.yml` with `persist-credentials: false` on checkout
- **Secret Protection:** `Internal Repo → Settings → Advanced Security` → **Secret Protection** → **Enable**, then **Push protection** → **Enable**
- **Review gate:** `Internal Repo → Settings → Rulesets → Rulesets → New ruleset → New branch ruleset` → **Require a pull request before merging**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| A separate public organization on GitHub.com (not in the EMU enterprise) and the target repository | Owner of that organization (personal account) | ☐ |
| A GitHub App owned by the public org with **Contents: Read & write**, installed on the target repo — or a fine-grained PAT / deploy key | Public org owner | ☐ |
| Store secrets, variables, and environments in the internal repository | Internal **repository administrator** | ☐ |
| GitHub Actions enabled on the internal repository | Org owner / repo admin | ☐ |
| Secret Protection (with push protection) on the internal repository | Repo admin / security team | ☐ |
| An approved open-source release process | Legal, security, engineering | ☐ |

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

*Workflow that pushes an approved branch from the internal repo to the public repo*

### Step A — Create the approval environment

**👤 Role:** Internal **repository administrator** · **📍 Portal:** GitHub

1. Internal repository → **Settings** → **Environments** → **New environment**.
2. Name it `public-release` and click **Configure environment**.
3. Select **Required reviewers**, add the release approvers, and click **Save protection rules**.
4. Under **Environment secrets** / **Environment variables**, add the app credentials from Step B.

### Step B — Add the workflow

Create this file in the **internal** repository:

```yaml
# .github/workflows/mirror-to-public.yml
name: Mirror to Public Repository

on:
  push:
    branches:
      - release/public

jobs:
  mirror:
    runs-on: ubuntu-latest
    environment: public-release        # pauses for an approver
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v6
        with:
          fetch-depth: 0               # full history
          persist-credentials: false   # don't send this repo's GITHUB_TOKEN to the public repo

      - name: Create a token for the public repo
        id: app-token
        uses: actions/create-github-app-token@v3
        with:
          client-id: ${{ vars.PUBLIC_APP_CLIENT_ID }}
          private-key: ${{ secrets.PUBLIC_APP_PRIVATE_KEY }}
          owner: your-public-org
          repositories: your-public-repo

      - name: Push to the public repository
        env:
          TOKEN: ${{ steps.app-token.outputs.token }}
        run: |
          git remote add public "https://x-access-token:${TOKEN}@github.com/your-public-org/your-public-repo.git"
          git push public HEAD:main --force
```

> ✅ **Result:** each push to `release/public` waits for approval, then mirrors to the public repository's `main` branch.

> ⚠️ **Warnings:**
> - `--force` overwrites the public branch. Treat the public repository as **read-only** — don't commit to it directly.
> - Without `persist-credentials: false`, `actions/checkout` keeps an auth header for `github.com` that overrides your token, and the push fails with a 403.
> - If the internal enterprise is on **GHE.com**, the checkout uses your GHE.com host and the push still targets `github.com`.

---

## 3️⃣ Authentication Options for Mirroring

| Method | Security | Scope | Best for |
|--------|----------|-------|----------|
| **GitHub App token** | Best — 1-hour tokens, not tied to a person | Specific permissions and repositories | Production mirroring (recommended) |
| **Deploy key** | Good | One repository | SSH-based mirroring without a user account |
| **Fine-grained PAT** | Basic — tied to a person, long-lived | Selected repositories | Short tests only |

### A) GitHub App (recommended)

**👤 Role:** Public **organization owner**

1. Public org → **Settings** → **Developer settings** → **GitHub Apps** → **New GitHub App**.
2. Name it, set the homepage URL, untick webhook **Active**, set **Repository permissions → Contents: Read & write**, choose **Only on this account**, and click **Create GitHub App**.
3. Copy the **Client ID**, then under **Private keys** click **Generate a private key**.
4. Click **Install App** → **Install** → **Only select repositories** → the public repo → **Install**.
5. In the **internal** repo's `public-release` environment, add variable `PUBLIC_APP_CLIENT_ID` and secret `PUBLIC_APP_PRIVATE_KEY` (the whole `.pem`).

### B) Deploy key

1. Generate a dedicated SSH key pair for mirroring.
2. Public repo → **Settings** → **Deploy keys** → **Add deploy key** → paste the **public** key → select **Allow write access** → **Add key**.
3. Store the **private** key as an environment secret in the internal repo and push over SSH (`git@github.com:your-public-org/your-public-repo.git`).

### C) Fine-grained PAT (testing only)

1. As an account in the public org: **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
2. **Resource owner:** the public org. **Repository access:** **Only select repositories** → the public repo. **Permissions:** **Contents: Read and write**.
3. Click **Generate token** (the org may require approval), then store it as an environment secret.

---

## 4️⃣ Selective Mirroring

*Mirror only what should be public — not everything*

| Technique | How |
|-----------|-----|
| **Mirror only a release branch** | Change the workflow trigger to a specific branch (e.g., `release/public`) |
| **Keep internal files out of the branch** | Never commit internal-only files to the mirror branch (`.gitignore` doesn't remove files already committed) |
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
| **Required PR approval for mirror branch** | A ruleset requiring pull request reviews on `release/public` |
| **Deployment approval** | The `public-release` environment with required reviewers |
| **Audit logs** | Review the enterprise audit log for workflow runs and environment approvals, and the public org's audit log for app activity |

### Enable Secret Protection on the source repo

**👤 Role:** **Repository administrator**

1. Internal repository → **Settings** → **Advanced Security** (under "Security and quality").
2. Next to **Secret Protection**, click **Enable** → **Enable Secret Protection**.
3. In the **Secret Protection** section, next to **Push protection**, click **Enable**.

### Require PR approval for the mirror branch

**👤 Role:** **Repository administrator**

1. Internal repository → **Settings** → **Rulesets** → **Rulesets** → **New ruleset** → **New branch ruleset**.
2. Name it, set **Enforcement status** to **Active**.
3. **Target branches** → **Add target** → **Include by pattern** → `release/public`.
4. Select **Require a pull request before merging** and set **Required approvals** to 1 or more.
5. Click **Create**.

> ✅ **Result:** nothing reaches the public mirror without a reviewed PR **and** an approved deployment to `public-release`.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Push to public mirror is denied** | The token, deploy key, or GitHub App installation lacks write access to the public target repository. | Grant only the required repository write permission, update the Actions secret, and test with a non-production branch first. |
| **Mirror workflow publishes more than intended** | The workflow mirrors all refs/history or runs on the wrong trigger. | Restrict triggers and refspecs, require PR approval before the mirror branch, and treat the public target as read-only. |
| **Secret scanning or push protection blocks the mirror** | The outgoing history contains a supported secret pattern. | Stop the workflow, revoke the exposed secret, remove it from history or choose a clean release branch, then rerun after security review. |
| **Public repository history is overwritten unexpectedly** | A force mirror push was used against a target that had independent commits. | Restore from the target repository backup or reflog if available, then enforce that the public target receives changes only from the mirror workflow. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: The mirror workflow is failing with authentication errors. What should I check?
**A:** First check that checkout uses `persist-credentials: false` — otherwise the internal repo's `GITHUB_TOKEN` header is sent to the public repo and you get a 403. Then confirm the app is installed on the public repo with **Contents: Read & write**, that `PUBLIC_APP_CLIENT_ID` / `PUBLIC_APP_PRIVATE_KEY` are available to the job (environment secrets need `environment: public-release`), and that `owner` and `repositories` match the public repo. For PATs or deploy keys, check they haven't expired or been removed.

---

### Q: Sensitive data was accidentally pushed to the public mirror. What should we do immediately?
**A:** Treat it as a security incident. Revoke and rotate the exposed credentials immediately — assume they're compromised. Pause the mirror workflow, rewrite the public history with `git filter-repo` and force-push (or make the repo private or delete it if the exposure is severe), and follow GitHub's "Removing sensitive data from a repository" guidance — contact GitHub Support to purge cached views and pull request refs. Then fix the internal branch before re-enabling the mirror.

---

### Q: The public mirror only shows the latest commit instead of the full history. What is wrong?
**A:** `actions/checkout` defaults to a shallow clone (`fetch-depth: 1`), and pushing a shallow clone to another repository either fails ("shallow update not allowed") or can't send the full history. Set `fetch-depth: 0` so the whole history is available before the push.

---

### Q: How do I stop mirroring to the public repository?
**A:** Disable the workflow (internal repo → **Actions** → the workflow → **⋯** → **Disable workflow**) or delete the file. Then remove access: uninstall the GitHub App from the public repo (or delete the deploy key, or revoke the PAT) and delete the stored credentials. The public repository keeps its current content but stops receiving updates.

---

### Q: Can I mirror only specific files or directories instead of the entire repository?
**A:** Yes. Keep a dedicated branch (such as `release/public`) that only contains approved content, and mirror just that branch. To publish a subdirectory or drop internal paths, run `git filter-repo` (for example `--path public-sdk/`) in the workflow on a fresh clone before pushing. `.gitignore` doesn't remove files that are already committed, so don't rely on it.

---

### Q: The mirror workflow runs but nothing changes in the public repo. What could be the issue?
**A:** If there are no new commits on the source branch since the last mirror run, `git push` will report "Everything up-to-date" and no changes will appear. Also verify the workflow trigger branch matches the branch you are committing to. Check the workflow run logs in the Actions tab for any error messages or skipped steps.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| EMU Dual Presence (Enterprise + Open Source) | `Identity/EMU Dual Presence (Enterprise + Open Source).md` |
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| GitHub App for CI/CD (No Seat Cost) | `Actions/GitHub App for CI-CD (No Seat Cost).md` |

---

## 📚 Resources

- [actions/checkout](https://github.com/actions/checkout)
- [actions/create-github-app-token](https://github.com/actions/create-github-app-token)
- [Making authenticated API requests with a GitHub App in a GitHub Actions workflow](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow)
- [Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)

---

*Last updated: October 2026*
